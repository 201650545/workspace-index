
### 音节级对齐器（问题一的核心）

文件：`\lib/training/reading_alignment.dart`

```dart
/// 整课朗读对齐器（**纯 Dart**：不依赖 Flutter、不碰麦克风）。
///
/// 2026-10-20 郭老师：
/// 　· 「以一篇课文为一个流程……把这个课文都读完，然后才结束」
/// 　· 「识别是实时的，我边说它就边识别」
///
/// 这份文件只解决一件事：**ASR 识别出来的字，怎么和课文对上**。
///
/// 为什么不能"一字对一字往后比"（老做法）：
/// 　· 漏读一个字 → 后面全部错位（"小蝌蚪**找**妈妈"读成"小蝌蚪妈妈"，
///    顺序比较会把"找"判成漏、"妈妈"判成错……全乱）；
/// 　· 课文里两次出现"妈妈" → 只能搜到第一个；
/// 　· ASR 下一次快照修正了文字 → 之前判好的字会反复变色。
///
/// 做法是**音节级序列对齐**（Needleman–Wunsch 变体）：
/// 　· 期望序列 = 课文的音节（去调），识别序列 = ASR 的音节（去调）；
/// 　· 打分：完全相同 3 分；同声母 1.5 分（读音接近，倾向判"可能读成"）；
///    其它 0.75 分（读到了别的音）；跳过（漏/多）各扣 1.2 分；
/// 　· 回溯得到每个课文音节的状态：匹配 / 可能读成 / 漏读。
///
/// 关键性质：**漏读只影响那一个**。回溯时"漏掉的那一格"与"多出来的那一格"
/// 若声母相同，就把"漏读"升级为"可能读成 X"（把听到的音记下来给孩子看）。
library;

/// 一个课文音节的判定结果。
enum ReadingTokenStatus {
  /// 还没读到。
  pending,

  /// 读到了，而且对上了。
  matched,

  /// 读到了，但不是这个音（听到了别的东西）—— 措辞要温和：不是"你错了"，
  /// 是"可能读成了 X"。
  substituted,

  /// 没听到（漏读）。
  missed,

  /// 听到了但还没定（暂定区：ASR 还在修正，不急着判）。
  uncertain,
}

/// 对齐后的一个课文音节。
class AlignedToken {
  const AlignedToken({
    required this.status,
    this.heard,
  });

  final ReadingTokenStatus status;

  /// 实际听到的音节（仅 [ReadingTokenStatus.substituted] / uncertain 有值）。
  final String? heard;

  bool get isSettled =>
      status == ReadingTokenStatus.matched ||
      status == ReadingTokenStatus.substituted ||
      status == ReadingTokenStatus.missed;
}

/// 一次对齐的结果。
class ReadingAlignment {
  const ReadingAlignment({
    required this.statuses,
    required this.heard,
    required this.matchedCount,
    required this.substitutedCount,
    required this.missedCount,
    required this.insertedCount,
    required this.score,
    this.frontier = 0,
  });

  /// 与期望序列等长。
  final List<ReadingTokenStatus> statuses;

  /// 与期望序列等长：听到的音节（没听到为 null）。
  final List<String?> heard;

  final int matchedCount;
  final int substitutedCount;
  final int missedCount;

  /// 识别序列里多出来的音节数（重复读、插话、杂音）。
  final int insertedCount;

  /// 对齐总分（越大越像）。
  final double score;

  /// **识别覆盖到的 expected 下标**（=「读到这儿了」）。
  ///
  /// 2026-10-21（流式实时反馈）：批式识别里，识别完了才判「漏读」；流式下
  /// 尾部那些**还没读到的**字如果也判 missed，孩子会看到满屏红字。
  /// ⇒ 尾部 `[frontier..statuses.length)` 应当保持 pending（还没读到），
  /// 只有真正被跳过的**中间**字才算漏读。
  ///
  /// 语义：[frontier] 是「最后一个 matched/substituted 的位置 + 1」；
  /// 若一个都没匹配上则为 0。
  final int frontier;
}

/// 音节级序列对齐器。
class ReadingAligner {
  const ReadingAligner();

  static const double _matchScore = 3.0;
  static const double _sameInitialScore = 1.5;
  static const double _otherScore = 0.75;
  static const double _gapPenalty = 1.2;

  /// 把 [recognized] 对齐到 [expected]。
  ///
  /// [expected] 与 [recognized] 都是**去调**音节（比较时用去调版；声母比较
  /// 也在去调版上做）。返回的 [ReadingAlignment.statuses] 与 [expected] 等长。
  ReadingAlignment align({
    required List<String> expected,
    required List<String> recognized,
  }) {
    final n = expected.length;
    final m = recognized.length;

    if (n == 0) {
      return ReadingAlignment(
        statuses: const [],
        heard: const [],
        matchedCount: 0,
        substitutedCount: 0,
        missedCount: 0,
        insertedCount: m,
        score: 0,
        frontier: 0,
      );
    }
    if (m == 0) {
      return ReadingAlignment(
        statuses: [for (var i = 0; i < n; i++) ReadingTokenStatus.missed],
        heard: [for (var i = 0; i < n; i++) null],
        matchedCount: 0,
        substitutedCount: 0,
        missedCount: n,
        insertedCount: 0,
        score: 0,
        frontier: 0,
      );
    }

    // dp[i][j] = 期望前 i 个 与 识别前 j 个 的最佳得分
    final dp = List.generate(
      n + 1,
      (_) => List<double>.filled(m + 1, 0),
    );
    for (var i = 1; i <= n; i++) {
      dp[i][0] = dp[i - 1][0] - _gapPenalty;
    }
    for (var j = 1; j <= m; j++) {
      dp[0][j] = dp[0][j - 1] - _gapPenalty;
    }
    for (var i = 1; i <= n; i++) {
      for (var j = 1; j <= m; j++) {
        final diag =
            dp[i - 1][j - 1] + _pairScore(expected[i - 1], recognized[j - 1]);
        final up = dp[i - 1][j] - _gapPenalty; // 期望多一个 → 漏读
        final left = dp[i][j - 1] - _gapPenalty; // 识别多一个 → 多读/插入
        dp[i][j] = _max3(diag, up, left);
      }
    }

    // 回溯
    final statuses = List<ReadingTokenStatus>.filled(n, ReadingTokenStatus.missed);
    final heard = List<String?>.filled(n, null);
    var inserted = 0;
    var i = n;
    var j = m;
    // 识别序列被耗尽的那一刻，expected 还剩 i 个 —— 那就是「读到这儿了」。
    //
    // 2026-10-21：流式识别的实时反馈需要这个。批式识别里，识别完了才判
    // 「漏读」；流式下，尾部那些**还没读到的**字如果也判成 missed，孩子会
    // 看到满屏红字。⇒ 尾部 [frontier..n) 应当保持 pending（还没读到），
    // 只有真正被跳过的**中间**字才算漏读。
    var frontier = n;
    while (i > 0 || j > 0) {
      if (i > 0 && j > 0) {
        final diag = dp[i - 1][j - 1] +
            _pairScore(expected[i - 1], recognized[j - 1]);
        if ((dp[i][j] - diag).abs() < 1e-9) {
          final e = expected[i - 1];
          final r = recognized[j - 1];
          if (e == r) {
            statuses[i - 1] = ReadingTokenStatus.matched;
            heard[i - 1] = r;
          } else {
            statuses[i - 1] = ReadingTokenStatus.substituted;
            heard[i - 1] = r;
          }
          i--;
          j--;
          continue;
        }
      }
      if (i > 0 && (dp[i][j] - (dp[i - 1][j] - _gapPenalty)).abs() < 1e-9) {
        // 期望多一个：先记成漏读，等回溯到"识别多出来"的那格时再升级
        statuses[i - 1] = ReadingTokenStatus.missed;
        i--;
        continue;
      }
      // 识别多一个（插入）：如果它和左边刚漏掉的那格声母相同，就把漏读升级
      // 成"可能读成"。
      if (j > 0) {
        inserted++;
        if (i > 0 && statuses[i - 1] == ReadingTokenStatus.missed) {
          final e = expected[i - 1];
          final r = recognized[j - 1];
          if (_sameInitial(e, r)) {
            statuses[i - 1] = ReadingTokenStatus.substituted;
            heard[i - 1] = r;
          }
        }
        j--;
      }
    }
    // 回溯走完时 j==0 ⇒ 识别序列耗尽于 i（若中间没有 gap 消耗，frontier 就是
    // 「识别覆盖到的 expected 长度」）。这里用「最后一次识别被消耗的位置」来
    // 精确计算：重新扫一遍 DP 最优路径不现实，改为用状态分布推断 ——
    // 尾部连续的 missed 就是「还没读到」。
    var covered = 0;
    for (var k = 0; k < n; k++) {
      if (statuses[k] == ReadingTokenStatus.matched ||
          statuses[k] == ReadingTokenStatus.substituted) {
        covered = k + 1;
      }
    }
    frontier = covered;

    var matched = 0;
    var substituted = 0;
    var missed = 0;
    for (final s in statuses) {
      switch (s) {
        case ReadingTokenStatus.matched:
          matched++;
        case ReadingTokenStatus.substituted:
          substituted++;
        case ReadingTokenStatus.missed:
          missed++;
        default:
          break;
      }
    }

    return ReadingAlignment(
      statuses: statuses,
      heard: heard,
      matchedCount: matched,
      substitutedCount: substituted,
      missedCount: missed,
      insertedCount: inserted,
      score: dp[n][m],
      frontier: frontier,
    );
  }

  static double _max3(double a, double b, double c) =>
      a > b ? (a > c ? a : c) : (b > c ? b : c);

  static double _pairScore(String expected, String recognized) {
    if (expected == recognized) return _matchScore;
    if (_sameInitial(expected, recognized)) return _sameInitialScore;
    return _otherScore;
  }

  /// 声母是否相同（"找" vs "爪" 算接近，"找" vs "猫" 不算）。
  static bool _sameInitial(String a, String b) {
    final ia = _initialOf(a);
    final ib = _initialOf(b);
    if (ia.isEmpty || ib.isEmpty) return false;
    if (ia == ib) return true;
    // zh/z、ch/c、sh/s 舌尖音互认（ASR 常见混淆）。
    const near = {
      'zh': 'z',
      'ch': 'c',
      'sh': 's',
    };
    return near[ia] != null && near[ia] == ib;
  }

  static const List<String> _initials = [
    'zh', 'ch', 'sh',
    'b', 'p', 'm', 'f', 'd', 't', 'n', 'l',
    'g', 'k', 'h', 'j', 'q', 'x',
    'r', 'z', 'c', 's', 'y', 'w',
  ];

  static String _initialOf(String syllable) {
    if (syllable.isEmpty) return '';
    for (final ini in _initials) {
      if (syllable.startsWith(ini)) return ini;
    }
    return '';
  }
}

```

### 拼音表结构（问题一核心）

文件：`lib/utils/chinese_pinyin.dart`（全文件 432KB，单字表由脚本生成，下面是结构与关键部分）

```dart
// Generated by tool/generate_pinyin_tool.py. DO NOT EDIT MANUALLY.


  // 词组表里所有含「长」的条目（可见：没有「长出」）：
    '长长的': ['cháng', 'cháng', 'de'],
    '长高': ['zhǎng', 'gāo'],
    '长长': ['cháng', 'cháng'],
    '长袍': ['cháng', 'páo'],
    '长短': ['cháng', 'duǎn'],
    '长江': ['cháng', 'jiāng'],
    '长大': ['zhǎng', 'dà'],
    '细长': ['xì', 'cháng'],
    '生长': ['shēng', 'zhǎng'],
```

### 口令检测（问题二）

文件：`\lib/training/wake_phrase.dart`

```dart
/// **口令（唤醒词）检测** —— 2026-10-21。
///
/// 郭老师的需求：
/// > 「给这个按钮增加实时识别功能。但为了避免它一直识别，我们需要设置一个
/// > 识别要求，即录制一个口令（口令可由用户自定义，比如"开始朗读课文"）。
/// > 当用户说出自定义口令，且系统准确识别到之后，才会触发正式测试。」
///
/// 做法：**流式识别 + 文本匹配**。
///
/// 为什么不用 sherpa 的 `KeywordSpotter`：
///   · 它要求 transducer 模型 + 另配一份含关键词的词表 ⇒ 用户每改一次口令
///     就要重生成词表、重新推模型；
///   · 我们已经有流式识别在跑（同一块音频、同一份模型），口令检测只是
///     **在已有的文本上做一次匹配**，零额外成本、零额外模型、任意自定义。
///
/// 归一化：ASR 输出里可能有标点、空白、全角/半角差异 ⇒ 匹配前先洗一遍。
library;

/// 把 ASR 文本归一化成「便于匹配口令」的形态。
///
/// 处理：
///   · 去掉所有标点（中英文）与空白
///   · 全角 → 半角（ＡＢＣ → ABC，数字同理）
///   · 统一小写
String normalizeForWakeMatch(String input) {
  final buffer = StringBuffer();
  for (final rune in input.runes) {
    // 全角 ASCII（！-～）→ 半角
    var c = rune;
    if (c >= 0xFF01 && c <= 0xFF5E) {
      c -= 0xFEE0;
    }
    if (c >= 0x3000 && c <= 0x303F) {
      continue; // CJK 标点
    }
    if (_isAsciiPunct(c)) continue;
    if (_isWhitespace(c)) continue;
    buffer.writeCharCode(c);
  }
  return buffer.toString().toLowerCase();
}

bool _isAsciiPunct(int c) =>
    (c >= 0x21 && c <= 0x2F) ||
    (c >= 0x3A && c <= 0x40) ||
    (c >= 0x5B && c <= 0x60) ||
    (c >= 0x7B && c <= 0x7E);

bool _isWhitespace(int c) =>
    c == 0x20 || c == 0x09 || c == 0x0A || c == 0x0D || c == 0x0A0;

// ─────────────────────────────────────────────────────────────
// 同音/近音容错表
// ─────────────────────────────────────────────────────────────

/// 口令里常见的**同音字**替换对：ASR 可能把「始」听成「使/市/是」这类。
///
/// 每一项 = [标准写法, 其它读法…, 同音字写法…]（单字候选放在最后）。
const List<List<String>> _homophoneGroups = [
  ['始', 'shi', '使', '市', '是', '驶', '式'],
  ['读', 'du', '独', '度', '渡', '毒'],
  ['课', 'ke', '刻', '科', '颗'],
  ['文', 'wen', '闻', '蚊'],
  ['开', 'kai', '该', '改'],
];

/// 口令判定结果。
enum WakeMatch {
  /// 没匹配上。
  none,

  /// **严格命中**：归一化文本里完整包含口令。
  exact,

  /// **宽松命中**：完整包含 + 允许一处同音字替换。
  loose,
}

/// 口令检测器。
///
/// 用法（流式）：
/// ```dart
/// final detector = WakePhraseDetector(phrase: '开始朗读');
/// stream.listen((text) {
///   final m = detector.check(text);
///   if (m != WakeMatch.none) { /* 触发正式测试 */ }
/// });
/// ```
///
/// 状态机：
///   · [check] 每次喂一段**累积文本**（流式识别吐的是累积结果）；
///   · 命中后 [consume] 把「已用于触发的部分」切掉，避免同一段音频二次触发；
///   · [reset] 换一句/换一次测试时清空。
class WakePhraseDetector {
  WakePhraseDetector({required String phrase})
      : _phrase = phrase,
        _normalized = normalizeForWakeMatch(phrase);

  final String _phrase;

  /// 归一化后的口令（匹配时用它）。
  final String _normalized;

  /// 归一化后的口令（公开给 UI 预览用）。
  String get normalized => _normalized;

  /// 原始口令（给 UI 显示用）。
  String get display => _phrase;

  bool _fired = false;

  /// 已经触发过（本次不再重复触发）。
  bool get fired => _fired;

  /// 累积文本里「已触发过的那一段」的结束位置（用于切掉）。
  int _consumedUpTo = 0;

  /// 检查一段累积文本。
  ///
  /// [text] 是流式识别的**累积**结果（越往后越长）。
  WakeMatch check(String text) {
    if (_fired) return WakeMatch.none;
    final norm = normalizeForWakeMatch(text);
    if (norm.isEmpty || _normalized.isEmpty) return WakeMatch.none;

    // 严格包含。
    final idx = norm.indexOf(_normalized);
    if (idx >= 0) {
      _fired = true;
      _consumedUpTo = idx + _normalized.length;
      return WakeMatch.exact;
    }
    // 宽松：允许一处同音字替换。
    if (_looseHit(norm)) {
      _fired = true;
      _consumedUpTo = norm.length;
      return WakeMatch.loose;
    }
    return WakeMatch.none;
  }

  /// 宽松匹配：把口令的每个字换成一个同音候选后看能否包含。
  bool _looseHit(String norm) {
    final chars = _normalized.split('');
    // 口令很短（≤6 字）时，枚举 1 处替换是可行的。
    for (var i = 0; i < chars.length; i++) {
      for (final alt in _alternativesFor(chars[i])) {
        final candidate = chars.toList()..[i] = alt;
        if (norm.contains(candidate.join())) return true;
      }
    }
    return false;
  }

  /// 某个字的同音候选。
  List<String> _alternativesFor(String ch) {
    final out = <String>[];
    for (final group in _homophoneGroups) {
      // group = [标准写法, 拼音写法…, 同音字写法…]
      final std = group.first;
      if (!std.contains(ch)) continue;
      for (final alt in group.skip(1)) {
        if (alt.length == 1) out.add(alt);
      }
    }
    return out;
  }

  /// 切掉「已用于触发」的那一段，返回**剩下的**文本（正式朗读的起点）。
  ///
  /// 口令说完之后，孩子接着读课文 ⇒ 识别的累积文本里，口令之后的部分才是
  /// 课文内容。切掉它，正式判定才不会被口令的字污染。
  String consume(String text) {
    if (_consumedUpTo <= 0) return text;
    final norm = normalizeForWakeMatch(text);
    if (_consumedUpTo >= norm.length) return '';
    return norm.substring(_consumedUpTo);
  }

  /// 清空（换一次测试 / 复位）。
  void reset() {
    _fired = false;
    _consumedUpTo = 0;
  }
}

```

### 跟读控制器 / 阶段机（问题二）

文件：`\lib/training/live_reading_controller.dart`

```dart
import 'dart:async';

import 'package:flutter/foundation.dart';

import '../creation/streaming_asr_engine.dart';
import '../preferences/app_preferences.dart';
import '../utils/chinese_pinyin.dart';
import 'continuous_reading_session.dart';
import 'read_along_session.dart';
import 'wake_phrase.dart';

/// 实时朗读的**阶段**。
enum LiveReadPhase {
  /// 还没开始（麦克风未开）。
  idle,

  /// 麦克风开着，正在**等口令**（此时识别文本不参与判分）。
  waitingWake,

  /// 口令已识别，正在正式判这一句（逐字上色）。
  reading,

  /// 这一句读完，等孩子点/自动进下一句。
  sentenceDone,
}

/// 实时朗读控制器（2026-10-21）。
///
/// 把三件事串起来：
///   ① **流式识别**（[StreamingAsrSession]）—— 边说边出字；
///   ② **口令触发**（[WakePhraseDetector]）—— 说出自定义口令才正式开始；
///   ③ **逐字对齐**（[ContinuousReadingSession.applyRecognition] 的 live 模式）
///      —— 读到哪哪儿就变绿，不再是「读完一句忽然整句变红」。
///
/// 纯逻辑（不碰 Flutter widget），可以在电脑上完整单测。
class LiveReadingController extends ChangeNotifier {
  LiveReadingController({
    required this.session,
    required this.streamingEngine,
    String wakePhrase = kDefaultReadWakePhrase,
  }) : _wake = WakePhraseDetector(phrase: wakePhrase);

  /// 整课朗读会话（提供 token 与对齐）。
  final ContinuousReadingSession session;

  /// 流式识别引擎。
  final StreamingAsrEngine streamingEngine;

  final WakePhraseDetector _wake;

  StreamingAsrSession? _asr;
  StreamSubscription<String>? _textSub;

  LiveReadPhase _phase = LiveReadPhase.idle;
  LiveReadPhase get phase => _phase;

  /// 当前口令（用户可改）。
  String get wakePhrase => _wake.display;

  /// 口令文本（流式识别吐出来的累积文本）。
  String _liveText = '';
  String get liveText => _liveText;

  /// 口令已识别（宽松命中也算）。
  WakeMatch? _wakeHit;
  WakeMatch? get wakeHit => _wakeHit;

  /// 口令触发后，正式朗读部分的文本（已切掉口令）。
  String _readingText = '';
  String get readingText => _readingText;

  /// 识别文本 → 音节（去调）。ASR 出的是汉字，这里按课文的音节表转。
  ///
  /// ⚠️ 中文 CTC/流式模型吐的是**汉字**（不是拼音），而对齐器比的是**音节**
  /// （去调拼音）。所以要把识别到的汉字转成音节才能比。
  ///
  /// 做法：识别文本的每个字，用课文的 `char → toneless 音节` 表查（同一课里
  /// 同一个字读音基本一致；多音字取课文里的那个读音 —— 这正是我们要的：
  /// 孩子在照课文读）。
  Map<String, String> _charToSyllable = {};

  /// 识别文本里**不在课文里**的字（无法转音节）—— 记下来做诊断。
  int _unknownChars = 0;

  /// 当前累积文本里是否**还带着口令**。
  ///
  /// 逐句模式的流程：
  ///   · 口令触发那一刻：文本 = 「口令 + 本句」⇒ consume 切掉口令；
  ///   · 这一句读全绿 ⇒ **重置流式** ⇒ 之后吐的文本 = 纯下一句 ⇒ 不再 consume。
  bool _textHasWake = false;

  /// 建立课文「字 → 音节」表。
  void _buildCharMap() {
    _charToSyllable = {};
    for (final t in session.tokens) {
      _charToSyllable.putIfAbsent(t.char, () => t.toneless);
    }
  }

  /// 开始实时朗读：开麦克风、等口令。
  Future<void> start() async {
    if (_phase != LiveReadPhase.idle) return;
    _buildCharMap();
    try {
      _asr = await streamingEngine.openSession(language: 'zh');
    } catch (error) {
      // 没有流式模型 ⇒ 降级（调用方改走离线转写链路）。
      // ignore: avoid_print
      print('[LiveRead] 流式引擎不可用：$error');
      _phase = LiveReadPhase.idle;
      notifyListeners();
      return;
    }
    _textSub = _asr!.texts.listen(_onText);
    _phase = LiveReadPhase.waitingWake;
    notifyListeners();
  }

  /// 停：关麦克风、关流式会话。
  Future<void> stop() async {
    _textSub?.cancel();
    _textSub = null;
    _asr?.dispose();
    _asr = null;
    _phase = LiveReadPhase.idle;
    notifyListeners();
  }

  /// 把一块 PCM 推给流式引擎（由录音器的 onPcm 回调驱动）。
  void pushPcm(List<int> pcm16k) {
    final asr = _asr;
    if (asr == null) {
      _pcmDropped++;
      return;
    }
    _pcmPushed++;
    if (_pcmPushed == 1 || _pcmPushed % 50 == 0) {
      // ignore: avoid_print
      print('[LiveRead] pcm pushed=$_pcmPushed bytes=${pcm16k.length}');
    }
    asr.pushPcm(Uint8List.fromList(pcm16k));
  }

  /// 推给引擎的 PCM 块数（诊断用）。
  int _pcmPushed = 0;

  /// 因引擎没开而丢掉的 PCM 块数（诊断用）。
  int _pcmDropped = 0;

  /// 推给引擎的 PCM 块数（诊断用）。
  int get pcmPushed => _pcmPushed;

  /// 丢掉的 PCM 块数（诊断用）。
  int get pcmDropped => _pcmDropped;

  /// 收到的识别文本条数（诊断用）。
  int _textCount = 0;

  /// 收到的识别文本条数（诊断用）。
  int get textCount => _textCount;

  /// 流式识别吐回来的**累积文本**。
  void _onText(String text) {
    _liveText = text;
    _textCount++;
    if (_textCount == 1 || _textCount % 20 == 0) {
      // ignore: avoid_print
      print('[LiveRead] text#$_textCount = "$text"');
    }

    if (_phase == LiveReadPhase.waitingWake) {
      final hit = _wake.check(text);
      if (hit == WakeMatch.none) {
        notifyListeners();
        return;
      }
      _wakeHit = hit;
      // 切掉口令，剩下的才是课文。
      _readingText = _wake.consume(text);
      _textHasWake = true;
      _phase = LiveReadPhase.reading;
      // ⚠️ 口令和课文可能在**同一段**识别结果里（孩子连着说）。
      //    这里必须立刻对齐一次，否则这段课文要等下一次文本更新才判
      //    —— 表现为「说完口令+课文，那句还是没上色」。
      _applyLive(_readingText);
      notifyListeners();
      return;
    }

    if (_phase == LiveReadPhase.reading) {
      // 口令之后的部分（累积）。
      //
      // ⚠️ 逐句模式：这一句读全绿后上层会**重置流式**（`_asr.reset()`），
      //    重置后吐的文本 = 纯下一句的内容 ⇒ 这时**不能再 consume**（否则会把
        //    下一句的头几个字切掉）。用 [_textHasWake] 区分这两种情况。
      _readingText = _textHasWake ? _wake.consume(text) : text;
      if (_textHasWake) {
        // 本句还没全绿 ⇒ 文本仍带着口令（重置前）。
      }
      final completed = _applyLive(_readingText);
      if (completed) {
        // 这一句全绿了 ⇒ 重置流式，让下一句从干净的音频重新开始。
        //
        // 不重置的话，流式累积文本会一直带着这一句 + 下一句，对齐到下一句时
        // 整体错位 —— 表现为「还没读完这句就一直往下跳」。
        _asr?.reset();
        _textHasWake = false;
        _readingText = '';
      }
      notifyListeners();
    }
  }

  /// 把识别文本喂给对齐器（live 模式 ⇒ 逐字推进）。
  ///
  /// 返回：这一句是否刚刚**全绿完成**（逐句模式）。
  bool _applyLive(String text) {
    if (text.isEmpty) return false;
    final syllables = <String>[];
    _unknownChars = 0;

    // 逐字取读音：
    //   · 课文里有的字 ⇒ 用**课文里的读音**（孩子在照课文读，多音字也对得上）
    //   · 课文里没有的字 ⇒ 用**拼音库给它的读音**（孩子读成了别的字）
    //     ⚠️ 绝不能**丢掉**！丢了的话，对齐器那边就变成「那个字没读到」
    //        （missed / pending），而不是「可能读成了 X」（substituted）——
    //        孩子看不到自己读错成了什么。
    for (final token in ChinesePinyin.format(text)) {
      if (!token.isChinese) continue; // 标点/空白不参与判定
      final inText = _charToSyllable[token.char];
      if (inText != null) {
        syllables.add(inText);
      } else {
        _unknownChars++;
        final py = token.pinyin;
        if (py != null && py.isNotEmpty) {
          syllables.add(ReadAlongSession.stripTones(py).toLowerCase());
        }
      }
    }
    if (syllables.isEmpty) return false;
    return session.applyRecognition(
      syllables: syllables,
      text: text,
      live: true,
    );
  }

  /// 识别文本里不在课文里的字数（诊断用：非零说明 ASR 听到了课文外的东西）。
  int get unknownChars => _unknownChars;

  @override
  void dispose() {
    _textSub?.cancel();
    _asr?.dispose();
    super.dispose();
  }
}

```

### 录音器：接口与 RecordConfig（问题三）

文件：`\lib/training/read_along_recorder.dart`

```dart
/// 跟读验收录音器（WP-H 阶段 1：收音 + 临时 wav + 能量法端点检测）。
///
/// 设计依据 `05_任务书\WP-H_跟读验收模式_可执行实现方案_2026-10-10.md`：
///   * §二 关键架构洞察：`AsrEngine.transcribe` 吃 wav 文件路径，不吃实时流
///     ——录完一句存个临时 wav 喂给引擎即可，不需要流式识别。
///   * §三 判读完：尾部静音 ≥ 600ms 判定孩子读完；录音兜底上限 = 该句音频
///     时长 × 3 + 2000ms；全程没检出语音不算失败。
///   * §4.5 端点检测：能量法 RMS（不引模型），阈值 0.02 满量程保守起步，
///     **真机再调**。
///   * §九.6：临时 wav 识别完即删——不长期留存、不进 SQLite、不进备份导出、
///     不进 Anki。
///   * §九.5：录音前播放侧已 pause（门在 `sentence_player_controller`），
///     外放不会录进自己。
///
/// **2026-10-13 采样率根因（修"只吐 1 个音节 ii / 0.05 分"）**：
/// 安卓 `AudioRecord` 只把请求的 sampleRate 当建议，设备按原生率交付
/// （小米14 = 48000）且不告知真实值。此前请求 16k 拿到的是 16k 头 +
/// 48k 身的 PCM：声音被拉慢 3 倍、频谱下移 3 倍（实测 91% 能量 <1kHz、
/// 峰值 290Hz），CTC 模型听到一滩糊声 → 整句只解出 "ii"。
/// 修法：明确请求 48000 + 管线内 [PcmDecimator] 低通抽取到 16k，
/// 采样率从此确定、可测。别再退回"请求 16k 赌设备"。
library;

import 'dart:async';
import 'dart:io';
import 'dart:math' as math;
import 'dart:typed_data';

import 'package:flutter/foundation.dart';
import 'package:path_provider/path_provider.dart';
import 'package:record/record.dart';

/// 麦克风权限被拒（方案 §三：降级「仅跟读不评分」，绝不死锁）。
class ReadAlongMicPermissionDenied implements Exception {
  const ReadAlongMicPermissionDenied();

  @override
  String toString() => '麦克风权限被拒：跟读验收降级为仅跟读不评分';
}

/// 收音事件（UI 只跟这一个流）。
sealed class ReadAlongRecorderEvent {}

/// 音量级（RMS 满量程 0~1）：驱动录音态脉冲圈。
final class RecorderLevel extends ReadAlongRecorderEvent {
  RecorderLevel(this.rms);

  final double rms;
}

/// 收音结束，临时 wav 已落盘（用完即删，见 [ReadAlongRecorder.deleteWav]）。
final class RecorderStopped extends ReadAlongRecorderEvent {
  RecorderStopped(this.wavPath, {required this.timedOut});

  final String wavPath;

  /// 是否触及兜底上限强制收尾（孩子一直不吭声/环境噪音挂住）。
  final bool timedOut;
}

/// 全程没检出语音（方案 §三「噪音/没读」行）：不算失败、不计重试次数，
/// 提示「没听到你，再来一次」。空文件已删，不会进判分链路。
final class RecorderSilent extends ReadAlongRecorderEvent {}

/// 收音链路出错（采集流中断等）：不死锁，回到「再来一次」。
final class RecorderFailed extends ReadAlongRecorderEvent {
  RecorderFailed(this.error);

  final Object error;
}

/// 麦克风抽象——录音包的接缝，测试注入假实现。
abstract class ReadAlongMic {
  /// 检查并（按平台策略）请求录音权限。false = 被拒。
  Future<bool> hasPermission();

  /// 开始 PCM16 单声道 16kHz 块流。
  Future<Stream<Uint8List>> startStream();

  /// 正常收尾（保留已采数据）。
  Future<void> stop();

  /// 丢弃本次收音。
  Future<void> cancel();

  Future<void> dispose();
}

/// 真实实现：record 包的 [AudioRecorder]。
class RecordMic implements ReadAlongMic {
  RecordMic([AudioRecorder? recorder]) : _recorder = recorder ?? AudioRecorder();

  final AudioRecorder _recorder;

  @override
  Future<bool> hasPermission() => _recorder.hasPermission();

  @override
  Future<Stream<Uint8List>> startStream() => _recorder.startStream(
        const RecordConfig(
          // 48000 主动请求（2026-10-13 真机结论）：安卓 AudioRecord 只把
          // sampleRate 当"请求值"，设备可能按原生率交付（小米14 = 48k）且
          // 不告诉你真实值。请求 16k 会拿到 16k 头 + 48k 身的 PCM，
          // 声音被拉慢 3 倍，ASR 只能吐乱码。主动要 48k、自己抽取到 16k，
          // 采样率从此是确定的，不再赌设备。
          encoder: AudioEncoder.pcm16bits,
          sampleRate: 48000,
          numChannels: 1,
          bitRate: 16000,
          // 真机调参（2026-10-11 小米14 实测）：孩子俯身对着放桌上的手机
          // 读，原始电平偏弱——自动增益把轻声抬起来，噪声抑制压环境底噪。
          autoGain: true,
          noiseSuppress: true,
          echoCancel: false,
        ),
      );

  @override
  Future<void> stop() async {
    await _recorder.stop();
  }

  @override
  Future<void> cancel() async {
    await _recorder.cancel();
  }

  @override
  Future<void> dispose() async {
    await _recorder.dispose();
```

### 录音器：PCM 抽取与旁路推送（问题三）

文件：`\lib/training/read_along_recorder.dart`

```dart
  /// 设备实际交付的采样率（2026-10-13 小米14 实测：请求 16k 也交付 48k，
  /// AudioRecord 不保证请求值）。进管线前先经 [PcmDecimator] 抽到
  /// [sampleRate]——下游（wav 头 / RMS / 时长）一律按 16k 算。
  final int inputSampleRate;

  final _events = StreamController<ReadAlongRecorderEvent>();

  /// UI 订阅这一个流：音量级 / 结束 / 静音 / 失败。
  Stream<ReadAlongRecorderEvent> get events => _events.stream;

  final _wav = ReadAlongWavBuilder();

  /// 连续收音模式用的**滑动窗口**环形缓冲（整课一直开着时）。
  PcmRingBuffer? _ring;
  SilenceTracker? _tracker;

  /// 本次收音的兜底上限（_onChunk 用它判超时，_finish 用它写诊断日志）。
  int _maxDurationMs = 0;
  PcmDecimator? _decimator;
  StreamSubscription<Uint8List>? _sub;
  String? _lastWavPath;
  bool _running = false;
  bool _finishing = false;

  /// 连续模式：静音**不**结束收音（整课一个流程，麦克风一直开着）。
  bool _continuous = false;

  bool get isRecording => _running;

  /// 目前已检出过语音（边读边显示的门：没声时不空跑一次解码）。
  bool get hasSpeechSoFar => _tracker?.heardAnySpeech ?? false;

  /// 开始一次跟读收音。权限被拒抛 [ReadAlongMicPermissionDenied]——
  /// 上层接到它就走「仅跟读不评分」降级，页面不死锁。
  ///
  /// [maxDurationMs] 兜底上限 = 该句音频时长 × 3 + 2000ms（方案 §三）。
  ///
  /// [continuous] = 整课连续收音（2026-10-20 读页重构）：静音**不**结束录音，
  /// 麦克风一直开着；识别改走 [snapshotWindowWav] 的滑动窗口。
  /// [windowBytes] 是那个窗口的字节容量（16kHz×2byte = 32KB/秒）。
  Future<void> start({
    required int maxDurationMs,
    bool continuous = false,
    int windowBytes = 0,
  }) async {
    if (_running) return;
    if (!await _mic.hasPermission()) {
      throw const ReadAlongMicPermissionDenied();
    }
    _finishing = false;
    _continuous = continuous;
    _tracker = SilenceTracker(
      windowMs: silenceWindowMs,
      threshold: rmsThreshold,
      sampleRate: sampleRate,
    );
    _ring = continuous && windowBytes > 0 ? PcmRingBuffer(windowBytes) : null;
    _maxDurationMs = maxDurationMs;
    _decimator = PcmDecimator(
      inputRate: inputSampleRate,
      outputRate: sampleRate,
    );
    final stream = await _mic.startStream();
    _sub = stream.listen(
      (chunk) => _onChunk(chunk, maxDurationMs),
      onDone: () => _finish(timedOut: false),
      onError: (Object error) {
        _fail(error);
      },
      cancelOnError: true,
    );
    _running = true;
  }

  void _onChunk(Uint8List chunk, int maxDurationMs) {
    final tracker = _tracker;
    if (!_running || _finishing || tracker == null) return;
    // 48k 设备 PCM → 16k（低通+抽取）再进 wav/端点检测，下游只认 16k。
    var readDone = false;
    for (final pcm16 in _decimator?.push(chunk) ?? const <Uint8List>[]) {
      _wav.add(pcm16);
      _ring?.add(pcm16);
      // 流式识别的旁路（2026-10-21）：同一块 16k PCM 也直接推给流式引擎。
      // 放在这里 ⇒ 与端点检测、wav 落盘**完全解耦**，旧的离线链路不受影响。
      onPcm?.call(pcm16);
      if (tracker.feed(pcm16)) readDone = true;
    }
    if (tracker.lastRms > 0 || readDone) {
      _events.add(RecorderLevel(tracker.lastRms));
    }
    if (readDone && !_continuous) {
      _finish(timedOut: false);
    } else if (tracker.elapsedMs >= maxDurationMs) {
      _finish(timedOut: true);
    }
  }

  Future<void> _finish({required bool timedOut}) async {
    if (_finishing) return;
    _finishing = true;
    _running = false;
```

### 会话层：expected/heard 怎么来的（问题一）

文件：`\lib/training/continuous_reading_session.dart`

```dart
    );
  }

  /// 建整篇 token 索引。
  ///
  /// 词组归属用 [ChineseWordSegmenter] 的切分结果（它按小学高频词表贪心
  /// 切词），但**判定单位是音节**（一个字一个音节）——与拼音排版的逐字上色
  /// 一致。
  static List<ReadingToken> buildTokens(List<Sentence> sentences) {
    final tokens = <ReadingToken>[];
    var global = 0;
    for (var s = 0; s < sentences.length; s++) {
      final text = sentences[s].chinese;
      if (text.trim().isEmpty) continue;
      final pinyinTokens = ChinesePinyin.format(text);
      final words = ChineseWordSegmenter.segment(text);
      final tokenToWord = <int, int>{};
      for (var w = 0; w < words.length; w++) {
        for (final ci in words[w].charIndices) {
          tokenToWord[ci] = w;
        }
      }
      for (var t = 0; t < pinyinTokens.length; t++) {
        final pt = pinyinTokens[t];
        final py = pt.pinyin;
        if (py == null || py.isEmpty) continue; // 标点：不参与判定
        tokens.add(ReadingToken(
          index: global++,
          sentenceIndex: s,
          charIndex: t,
          char: pt.char,
          pinyin: py,
          toneless: ReadAlongSession.stripTones(py).toLowerCase(),
          wordKey: '${s}#${tokenToWord[t] ?? t}',
        ));
      }
```

### 会话层：流式对齐的基准与音节序列（问题一）

文件：`\lib/training/continuous_reading_session.dart`

```dart
    var sentenceEnd = _committedCursor;
    while (sentenceEnd < _tokens.length &&
        _tokens[sentenceEnd].sentenceIndex == sentenceIndex) {
      sentenceEnd++;
    }

    // 对齐的**基准起点**。
    //
    // ⚠️ 流式（live）与批式的基准不一样，这是 2026-10-21 踩出来的坑：
    //   · 批式：一次喂「这一句剩下的部分」⇒ 基准 = _committedCursor（句中）。
    //   · 流式：每次喂的是**从这一句开头起的累积文本**（'池' → '池塘' →
    //     '池塘里'）⇒ 基准必须是**句首**，否则第二次对齐时 expected 已经被
    //     cursor 截短，识别的累积文本却仍从句首开始 ⇒ 整体错位
    //     （「塘」被配到「chi」上，判成读错）。
    var base = _committedCursor;
    if (live) {
      if (_mode == ReadingMode.fullText) {
        base = 0; // 全文模式：累积文本对应全篇开头。
      } else {
        base = _committedCursor;
        while (base > 0 && _tokens[base - 1].sentenceIndex == sentenceIndex) {
          base--;
        }
      }
    }

    // 期望范围：
    //  · 句为单位 ⇒ 只对齐**当前句**剩下的音节（读完这句再进下一句）。
    //  · 全文     ⇒ 对齐**整篇剩下的**音节（读完整篇再统一处理）。
    final int alignEnd;
    if (_mode == ReadingMode.fullText) {
      alignEnd = _tokens.length;
    } else {
      alignEnd = sentenceEnd;
    }

    final expected = [
      for (var i = base; i < alignEnd; i++) _tokens[i].toneless,
    ];
    final heard = [
      ..._carry,
      for (final s in syllables)
        if (s.isNotEmpty) ReadAlongSession.stripTones(s).toLowerCase(),
    ];
    _carry = const [];

    final result = _aligner.align(expected: expected, recognized: heard);

    // 写回状态：只动当前句（已确认的前缀不改 —— 红绿不闪）。
    var contiguous = base;
    for (var k = 0; k < result.statuses.length; k++) {
      final idx = base + k;
      if (idx >= _tokens.length) break;
      var status = result.statuses[k];

      // ⚠️ 流式（live）：尾部那些**还没读到**的字不能判「漏读」。
      //
      // 批式识别是「读完一句再判」，尾部判 missed 是对的（真的漏了）。
      // 流式下只识别到句中，尾部只是**还没读**——判 missed 会让满屏红字。
      // ⇒ 用对齐器的 frontier（识别覆盖到的位置）把尾部还原成 pending。
      if (live && k >= result.frontier) {
        status = ReadingTokenStatus.pending;
      }

      if (idx < _committedCursor) continue;
      _tokenStates[idx] = status;
      if (status != ReadingTokenStatus.pending) {
        _tokenHeard[idx] = result.heard[k];
      }
      // ⚠️ 只有**完全对上**（matched）才推进游标。
      //
```
