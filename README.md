# shichu-appimport React, { useMemo, useState } from "react";

/*
  四柱推命アプリ App.jsx 完全単体版
  --------------------------------------------------
  Vite React の src/App.jsx を、このコードで全部置き換えてください。

  起動方法:
  npm create vite@latest . -- --template react
  npm install
  npm run dev

  このアプリでできること:
  - 生年月日と出生時刻から四柱を出す
  - 年柱、月柱、日柱、時柱を表示
  - 通変星、十二運、空亡、五行バランスを表示
  - 弟さんが迷わないように、使い方と読み方の説明を表示

  暦計算:
  - 年柱: 立春、太陽黄経315度で切替
  - 月柱: 節入り、太陽黄経315度から30度ごとで切替
  - 日柱: ユリウス日番号から六十干支を算出
  - 時柱: 日干と出生時刻から算出

  注意:
  - 太陽黄経は簡易天文式による近似計算です。
  - 身内・個人利用・プレゼント用として使いやすい構成です。
  - 有料鑑定で使う場合は、万年暦との照合をおすすめします。
*/

const STEMS = [
  { k: "甲", yy: "陽", e: "木", img: "大樹", text: "まっすぐで向上心が強く、自分の軸を持つほど運が開く人です。" },
  { k: "乙", yy: "陰", e: "木", img: "草花", text: "柔軟で調和力があり、環境に合わせながら魅力を育てる人です。" },
  { k: "丙", yy: "陽", e: "火", img: "太陽", text: "明るく存在感があり、人を照らす発信力を持つ人です。" },
  { k: "丁", yy: "陰", e: "火", img: "灯火", text: "繊細で感性が鋭く、好きなことに集中すると才能が光る人です。" },
  { k: "戊", yy: "陽", e: "土", img: "山", text: "安定感と包容力があり、人から頼られやすい人です。" },
  { k: "己", yy: "陰", e: "土", img: "畑", text: "現実感覚があり、物事や人を丁寧に育てる人です。" },
  { k: "庚", yy: "陽", e: "金", img: "刀", text: "決断力があり、困難を突破する勝負強さを持つ人です。" },
  { k: "辛", yy: "陰", e: "金", img: "宝石", text: "繊細で品格があり、専門性や美意識を磨くほど輝く人です。" },
  { k: "壬", yy: "陽", e: "水", img: "海", text: "知性と自由さがあり、広い世界で可能性を広げる人です。" },
  { k: "癸", yy: "陰", e: "水", img: "雨", text: "洞察力があり、静かに本質を見抜く力を持つ人です。" },
];

const BRANCHES = [
  { k: "子", animal: "鼠", e: "水", hidden: ["癸"] },
  { k: "丑", animal: "牛", e: "土", hidden: ["己", "癸", "辛"] },
  { k: "寅", animal: "虎", e: "木", hidden: ["甲", "丙", "戊"] },
  { k: "卯", animal: "兎", e: "木", hidden: ["乙"] },
  { k: "辰", animal: "龍", e: "土", hidden: ["戊", "乙", "癸"] },
  { k: "巳", animal: "蛇", e: "火", hidden: ["丙", "庚", "戊"] },
  { k: "午", animal: "馬", e: "火", hidden: ["丁", "己"] },
  { k: "未", animal: "羊", e: "土", hidden: ["己", "丁", "乙"] },
  { k: "申", animal: "猿", e: "金", hidden: ["庚", "壬", "戊"] },
  { k: "酉", animal: "鶏", e: "金", hidden: ["辛"] },
  { k: "戌", animal: "犬", e: "土", hidden: ["戊", "辛", "丁"] },
  { k: "亥", animal: "猪", e: "水", hidden: ["壬", "甲"] },
];

const ELEMENTS = ["木", "火", "土", "金", "水"];
const GEN = { 木: "火", 火: "土", 土: "金", 金: "水", 水: "木" };
const CTRL = { 木: "土", 火: "金", 土: "水", 金: "木", 水: "火" };
const GEN_BY = Object.fromEntries(Object.entries(GEN).map(([a, b]) => [b, a]));
const CTRL_BY = Object.fromEntries(Object.entries(CTRL).map(([a, b]) => [b, a]));

const ELEMENT_INFO = {
  木: { icon: "🌳", good: "成長・学習・企画・教育", bad: "理想先行、頑固、焦り", lucky: "植物、朝の散歩、読書、緑色" },
  火: { icon: "🔥", good: "表現・人気・直感・発信", bad: "感情的、消耗、目立ちすぎ", lucky: "太陽、発信、赤色、笑う時間" },
  土: { icon: "⛰️", good: "安定・信用・管理・育成", bad: "抱え込み、停滞、慎重すぎ", lucky: "整理整頓、黄色、貯金、生活リズム" },
  金: { icon: "💎", good: "決断・専門性・整理・美意識", bad: "完璧主義、批判的、緊張", lucky: "白色、断捨離、上質な道具、資格" },
  水: { icon: "💧", good: "知性・情報・自由・柔軟性", bad: "考えすぎ、不安、流されやすい", lucky: "水辺、入浴、黒色、日記、学習" },
};

const TEN_GOD_TEXT = {
  比肩: "自立心と芯の強さ。自分の看板で勝負すると力が出ます。",
  劫財: "仲間運と巻き込み力。人と組むほど大きく動けます。",
  食神: "楽しみと表現の星。自然体の発信が運を呼びます。",
  傷官: "才能と感性の星。型にはまらない発想が武器です。",
  偏財: "人脈と商才の星。広く動くほどチャンスを拾えます。",
  正財: "堅実と信用の星。積み上げ型の金運があります。",
  偏官: "勝負と突破力の星。困難な場面で本領を発揮します。",
  正官: "品位と社会性の星。信頼される場所で評価されます。",
  偏印: "ひらめきと専門性の星。独自の学びが道を作ります。",
  印綬: "知性と保護の星。深く学ぶほど運が安定します。",
};

const PHASE = {
  甲: { 亥: "長生", 子: "沐浴", 丑: "冠帯", 寅: "建禄", 卯: "帝旺", 辰: "衰", 巳: "病", 午: "死", 未: "墓", 申: "絶", 酉: "胎", 戌: "養" },
  乙: { 午: "長生", 巳: "沐浴", 辰: "冠帯", 卯: "建禄", 寅: "帝旺", 丑: "衰", 子: "病", 亥: "死", 戌: "墓", 酉: "絶", 申: "胎", 未: "養" },
  丙: { 寅: "長生", 卯: "沐浴", 辰: "冠帯", 巳: "建禄", 午: "帝旺", 未: "衰", 申: "病", 酉: "死", 戌: "墓", 亥: "絶", 子: "胎", 丑: "養" },
  丁: { 酉: "長生", 申: "沐浴", 未: "冠帯", 午: "建禄", 巳: "帝旺", 辰: "衰", 卯: "病", 寅: "死", 丑: "墓", 子: "絶", 亥: "胎", 戌: "養" },
  戊: { 寅: "長生", 卯: "沐浴", 辰: "冠帯", 巳: "建禄", 午: "帝旺", 未: "衰", 申: "病", 酉: "死", 戌: "墓", 亥: "絶", 子: "胎", 丑: "養" },
  己: { 酉: "長生", 申: "沐浴", 未: "冠帯", 午: "建禄", 巳: "帝旺", 辰: "衰", 卯: "病", 寅: "死", 丑: "墓", 子: "絶", 亥: "胎", 戌: "養" },
  庚: { 巳: "長生", 午: "沐浴", 未: "冠帯", 申: "建禄", 酉: "帝旺", 戌: "衰", 亥: "病", 子: "死", 丑: "墓", 寅: "絶", 卯: "胎", 辰: "養" },
  辛: { 子: "長生", 亥: "沐浴", 戌: "冠帯", 酉: "建禄", 申: "帝旺", 未: "衰", 午: "病", 巳: "死", 辰: "墓", 卯: "絶", 寅: "胎", 丑: "養" },
  壬: { 申: "長生", 酉: "沐浴", 戌: "冠帯", 亥: "建禄", 子: "帝旺", 丑: "衰", 寅: "病", 卯: "死", 辰: "墓", 巳: "絶", 午: "胎", 未: "養" },
  癸: { 卯: "長生", 寅: "沐浴", 丑: "冠帯", 子: "建禄", 亥: "帝旺", 戌: "衰", 酉: "病", 申: "死", 未: "墓", 午: "絶", 巳: "胎", 辰: "養" },
};

const VOID = [
  { start: 0, v: ["戌", "亥"] },
  { start: 10, v: ["申", "酉"] },
  { start: 20, v: ["午", "未"] },
  { start: 30, v: ["辰", "巳"] },
  { start: 40, v: ["寅", "卯"] },
  { start: 50, v: ["子", "丑"] },
];

const TERMS = [
  { name: "立春", angle: 315, branch: 2, m: 2, d: 4 },
  { name: "啓蟄", angle: 345, branch: 3, m: 3, d: 6 },
  { name: "清明", angle: 15, branch: 4, m: 4, d: 5 },
  { name: "立夏", angle: 45, branch: 5, m: 5, d: 6 },
  { name: "芒種", angle: 75, branch: 6, m: 6, d: 6 },
  { name: "小暑", angle: 105, branch: 7, m: 7, d: 7 },
  { name: "立秋", angle: 135, branch: 8, m: 8, d: 8 },
  { name: "白露", angle: 165, branch: 9, m: 9, d: 8 },
  { name: "寒露", angle: 195, branch: 10, m: 10, d: 8 },
  { name: "立冬", angle: 225, branch: 11, m: 11, d: 7 },
  { name: "大雪", angle: 255, branch: 0, m: 12, d: 7 },
  { name: "小寒", angle: 285, branch: 1, m: 1, d: 6, next: true },
];

const mod = (n, m) => ((n % m) + m) % m;
const rad = (d) => (d * Math.PI) / 180;
const stem = (k) => STEMS.find((s) => s.k === k);

function pillar(si, bi) {
  return { si, bi, stem: STEMS[si], branch: BRANCHES[bi], label: STEMS[si].k + BRANCHES[bi].k };
}

function jd(date) {
  return date.getTime() / 86400000 + 2440587.5;
}

function jdn(y, m, d) {
  const a = Math.floor((14 - m) / 12);
  const yy = y + 4800 - a;
  const mm = m + 12 * a - 3;
  return d + Math.floor((153 * mm + 2) / 5) + 365 * yy + Math.floor(yy / 4) - Math.floor(yy / 100) + Math.floor(yy / 400) - 32045;
}

function sunLon(date) {
  const T = (jd(date) - 2451545.0) / 36525;
  const L0 = mod(280.46646 + 36000.76983 * T + 0.0003032 * T * T, 360);
  const M = mod(357.52911 + 35999.05029 * T - 0.0001537 * T * T, 360);
  const C =
    (1.914602 - 0.004817 * T - 0.000014 * T * T) * Math.sin(rad(M)) +
    (0.019993 - 0.000101 * T) * Math.sin(rad(2 * M)) +
    0.000289 * Math.sin(rad(3 * M));
  const omega = 125.04 - 1934.136 * T;
  return mod(L0 + C - 0.00569 - 0.00478 * Math.sin(rad(omega)), 360);
}

function angleDiff(lon, target) {
  return ((lon - target + 540) % 360) - 180;
}

function termTime(solarYear, t) {
  const y = t.next ? solarYear + 1 : solarYear;
  let left = new Date(y, t.m - 1, t.d - 6, 0, 0, 0);
  let right = new Date(y, t.m - 1, t.d + 6, 23, 59, 59);
  for (let i = 0; i < 55; i++) {
    const mid = new Date((left.getTime() + right.getTime()) / 2);
    if (angleDiff(sunLon(mid), t.angle) < 0) left = mid;
    else right = mid;
  }
  return right;
}

function solarYearOf(date) {
  const y = date.getFullYear();
  const lichun = termTime(y, TERMS[0]);
  return date >= lichun ? y : y - 1;
}

function yearPillar(date) {
  const y = solarYearOf(date);
  return pillar(mod(y - 4, 10), mod(y - 4, 12));
}

function monthPillar(date, yearStemIndex) {
  const sy = solarYearOf(date);
  const terms = TERMS.map((t) => ({ ...t, time: termTime(sy, t) })).sort((a, b) => a.time - b.time);
  let active = terms[0];
  for (const t of terms) {
    if (date >= t.time) active = t;
    else break;
  }
  const branchIndex = active.branch;
  const tigerStem = [2, 4, 6, 8, 0][Math.floor(yearStemIndex / 2)];
  const stemIndex = mod(tigerStem + mod(branchIndex - 2, 12), 10);
  return { ...pillar(stemIndex, branchIndex), termName: active.name, termTime: active.time };
}

function dayPillar(date) {
  const n = jdn(date.getFullYear(), date.getMonth() + 1, date.getDate());
  const cycle = mod(n - 11, 60);
  return pillar(cycle % 10, cycle % 12);
}

function hourPillar(date, dayStemIndex) {
  const branchIndex = mod(Math.floor((date.getHours() + 1) / 2), 12);
  const firstStem = [0, 2, 4, 6, 8][Math.floor(dayStemIndex / 2)];
  return pillar(mod(firstStem + branchIndex, 10), branchIndex);
}

function tenGod(dayStem, targetStem) {
  const same = dayStem.yy === targetStem.yy;
  if (dayStem.e === targetStem.e) return same ? "比肩" : "劫財";
  if (GEN[dayStem.e] === targetStem.e) return same ? "食神" : "傷官";
  if (CTRL[dayStem.e] === targetStem.e) return same ? "偏財" : "正財";
  if (CTRL_BY[dayStem.e] === targetStem.e) return same ? "偏官" : "正官";
  if (GEN_BY[dayStem.e] === targetStem.e) return same ? "偏印" : "印綬";
  return "-";
}

function voidBranches(dayStemIndex, dayBranchIndex) {
  const cycle = mod(dayStemIndex + ((dayBranchIndex - dayStemIndex + 12) % 12), 60);
  return (VOID.find((g) => cycle >= g.start && cycle < g.start + 10) || VOID[0]).v;
}

function score(pillars) {
  const c = Object.fromEntries(ELEMENTS.map((e) => [e, 0]));
  Object.values(pillars).forEach((p) => {
    c[p.stem.e] += 1.4;
    c[p.branch.e] += 1.0;
    p.branch.hidden.forEach((h, i) => {
      c[stem(h).e] += i === 0 ? 0.5 : 0.25;
    });
  });
  return c;
}

function strength(dayElement, counts, monthBranch) {
  const support = counts[dayElement] + counts[GEN_BY[dayElement]] * 0.8;
  const pressure = counts[GEN[dayElement]] * 0.5 + counts[CTRL[dayElement]] * 0.7 + counts[CTRL_BY[dayElement]] * 0.9;
  const season = monthBranch.e === dayElement || monthBranch.e === GEN_BY[dayElement] ? 1.2 : 0;
  const v = support + season - pressure;
  if (v >= 2.4) return { label: "身強", useful: [CTRL[dayElement], GEN[dayElement]], explanation: "日主の力が強め。才能を外に出すこと、行動して形にすることが開運です。" };
  if (v <= -0.8) return { label: "身弱", useful: [GEN_BY[dayElement], dayElement], explanation: "日主の力が弱め。まずは学び・休息・味方作りで自分を整えることが開運です。" };
  return { label: "中和", useful: [CTRL[dayElement], GEN_BY[dayElement]], explanation: "バランス型。攻める時と整える時を分けると運が安定します。" };
}

function buildChart(birthDate, birthTime) {
  const [y, m, d] = birthDate.split("-").map(Number);
  const [hh, mm] = birthTime.split(":").map(Number);
  const date = new Date(y, m - 1, d, hh, mm || 0, 0);
  const yp = yearPillar(date);
  const mp = monthPillar(date, yp.si);
  const dp = dayPillar(date);
  const hp = hourPillar(date, dp.si);
  const base = { 年柱: yp, 月柱: mp, 日柱: dp, 時柱: hp };
  const counts = score(base);
  const sorted = [...ELEMENTS].sort((a, b) => counts[b] - counts[a]);
  const st = strength(dp.stem.e, counts, mp.branch);
  const voids = voidBranches(dp.si, dp.bi);
  const pillars = Object.fromEntries(Object.entries(base).map(([name, p]) => {
    const hidden = p.branch.hidden.map((h) => ({ k: h, god: tenGod(dp.stem, stem(h)) }));
    return [name, { ...p, god: name === "日柱" ? "日主" : tenGod(dp.stem, p.stem), hidden, phase: PHASE[dp.stem.k]?.[p.branch.k] || "-", void: voids.includes(p.branch.k) }];
  }));
  return { date, pillars, dayMaster: dp.stem, counts, strongest: sorted[0], weakest: sorted[sorted.length - 1], strength: st, voids, solarYear: solarYearOf(date) };
}

function reading(chart, name, gender) {
  const n = name || "お客様";
  const dm = chart.dayMaster;
  const mainGod = chart.pillars.月柱.god;
  const phase = chart.pillars.日柱.phase;
  const strong = chart.strongest;
  const weak = chart.weakest;
  const tone = gender === "female" ? "受け取る力と感性" : gender === "male" ? "決断力と行動力" : "自然体の魅力";
  return [
    ["本質", "⭐", `${n}様の日主は「${dm.k}（${dm.img}）」。${dm.text} ${tone}を無理に作るより、本来の性質を整えることで運が強くなります。`],
    ["才能", "📜", `月柱の通変星は「${mainGod}」。${TEN_GOD_TEXT[mainGod] || "社会で出やすい才能を示します。"}`],
    ["十二運", "🌙", `日柱の十二運は「${phase}」。人生のエネルギーの出方を表します。自分に合う速度で運を使うことが大切です。`],
    ["五行", "⚖️", `五行では「${strong}」が強く、「${weak}」が不足しやすい命式です。強い五行は才能ですが、出すぎると「${ELEMENT_INFO[strong].bad}」になりやすいです。`],
    ["開運", "🎁", `命式は「${chart.strength.label}」寄り。喜神の目安は「${chart.strength.useful.join("・")}」。${chart.strength.useful.map((e) => ELEMENT_INFO[e].lucky).join("、")} を取り入れると整いやすいです。`],
    ["注意点", "🛡️", `空亡は「${chart.voids.join("・")}」。執着を手放すほど流れが良くなるテーマです。大きな判断は焦らず確認を重ねると安定します。`],
    ["仕事運", "💼", `${ELEMENT_INFO[strong].good}が強みです。得意なことを実績や形にして見せるほど評価されます。`],
    ["恋愛・人間関係", "💗", `日主を支える「${GEN_BY[dm.e]}」や、喜神目安の「${chart.strength.useful.join("・")}」の気を持つ相手と安心感が出やすいでしょう。`],
    ["金運", "💰", `金運は一発勝負より、得意な五行を仕事・信用・発信に変えることで伸びます。不足する「${weak}」を補う習慣が鍵です。`],
  ];
}

function shareText(chart, name, date, time) {
  return `${name || "お客様"}の四柱推命鑑定\n${date} ${time} 生まれ\n年柱:${chart.pillars.年柱.label}\n月柱:${chart.pillars.月柱.label}\n日柱:${chart.pillars.日柱.label}\n時柱:${chart.pillars.時柱.label}\n日主:${chart.dayMaster.k}（${chart.dayMaster.img}）\n身強身弱:${chart.strength.label}\n強い五行:${chart.strongest}\n補いたい五行:${chart.weakest}\n喜神目安:${chart.strength.useful.join("・")}\n空亡:${chart.voids.join("・")}`;
}

function PillarCard({ title, p }) {
  return <div className="card pillar">
    <div className="topline"><b>{title}</b>{p.void && <span>空亡</span>}</div>
    <div className="bigKanji">{p.label}</div>
    <div className="mini">
      <div><small>天干</small><b>{p.stem.k}・{p.stem.yy}{p.stem.e}</b></div>
      <div><small>地支</small><b>{p.branch.k}・{p.branch.animal}</b></div>
      <div><small>通変星</small><b>{p.god}</b></div>
      <div><small>十二運</small><b>{p.phase}</b></div>
    </div>
    <p className="muted small">蔵干: {p.hidden.map((h) => `${h.k}/${h.god}`).join("・")}</p>
    {title === "月柱" && <p className="muted small">節入り: {p.termName} {p.termTime?.toLocaleString()}</p>}
  </div>;
}

function HelpSection() {
  return <section className="helpGrid noPrint">
    <div className="card help"><h3>① まず入力する</h3><p>名前、生年月日、出生時刻を入れるだけで鑑定できます。出生時刻が分からない時は、いったん12:00で見てください。</p></div>
    <div className="card help"><h3>② 一番大事なのは日主</h3><p>日主はその人の中心です。性格・考え方・運の使い方を見る時は、まず日主を見ます。</p></div>
    <div className="card help"><h3>③ 月柱は才能と社会運</h3><p>月柱は仕事、才能、社会での出方を見ます。通変星がその人の使いやすい才能です。</p></div>
    <div className="card help"><h3>④ 五行でバランスを見る</h3><p>強い五行は才能。不足する五行は開運ポイントです。足りない五行を生活に取り入れると整いやすくなります。</p></div>
  </section>;
}

export default function App() {
  const [name, setName] = useState("お客様");
  const [birthDate, setBirthDate] = useState("1995-04-15");
  const [birthTime, setBirthTime] = useState("12:00");
  const [place, setPlace] = useState("東京");
  const [gender, setGender] = useState("none");
  const [copied, setCopied] = useState(false);

  const chart = useMemo(() => buildChart(birthDate, birthTime), [birthDate, birthTime]);
  const texts = useMemo(() => reading(chart, name, gender), [chart, name, gender]);
  const max = Math.max(...Object.values(chart.counts));

  async function copy() {
    const text = shareText(chart, name, birthDate, birthTime);
    try {
      await navigator.clipboard.writeText(text);
      setCopied(true);
      setTimeout(() => setCopied(false), 1400);
    } catch {
      alert(text);
    }
  }

  return <div className="app">
    <style>{css}</style>
    <header className="hero">
      <div><div className="badge">✨ 四柱推命 本格鑑定</div><h1>命式鑑定アプリ</h1><p>弟さんがそのまま占えるように、使い方と読み方の説明つきです。</p></div>
      <div className="actions noPrint"><button className="btn sub" onClick={copy}>{copied ? "コピー済み" : "結果をコピー"}</button><button className="btn" onClick={() => window.print()}>印刷 / PDF</button></div>
    </header>

    <HelpSection />

    <div className="layout">
      <aside className="card form noPrint">
        <h2>鑑定入力</h2>
        <label>名前</label><input value={name} onChange={(e) => setName(e.target.value)} />
        <label>生年月日</label><input type="date" value={birthDate} onChange={(e) => setBirthDate(e.target.value)} />
        <label>出生時刻</label><input type="time" value={birthTime} onChange={(e) => setBirthTime(e.target.value)} />
        <label>出生地</label><input value={place} onChange={(e) => setPlace(e.target.value)} />
        <label>性別表現</label><select value={gender} onChange={(e) => setGender(e.target.value)}><option value="none">指定なし</option><option value="female">女性</option><option value="male">男性</option></select>
        <div className="note"><b>使い方</b><br />入力を変えると自動で結果が変わります。結果をコピーすればLINEやメールに貼れます。</div>
      </aside>

      <main className="main">
        <section className="card summary"><div><span className="gold small">鑑定対象</span><h2>{name || "お客様"} 様</h2><p>{birthDate} {birthTime} 生まれ / 出生地: {place}</p><p className="muted small">暦年: {chart.solarYear}年 / 月柱切替: {chart.pillars.月柱.termName}</p></div><div className="master"><span>日主</span><b>{chart.dayMaster.k}</b><em>{chart.dayMaster.img}</em></div></section>

        <section className="explain card"><h2>この結果の見方</h2><p><b>日主</b>は本人の中心、<b>月柱</b>は才能と社会運、<b>五行バランス</b>は強みと不足、<b>喜神</b>は開運に使いやすい五行、<b>空亡</b>は執着を手放すテーマとして見ます。</p></section>

        <section className="pillars">{Object.entries(chart.pillars).map(([k, p]) => <PillarCard key={k} title={k} p={p} />)}</section>

        <section className="two"><div className="card"><h2>五行バランス</h2>{ELEMENTS.map((e) => <div className="barbox" key={e}><div className="row"><b>{ELEMENT_INFO[e].icon} {e}</b><span>{chart.counts[e].toFixed(1)}</span></div><div className="bar"><div style={{ width: `${Math.max(7, chart.counts[e] / max * 100)}%` }} /></div><p className="muted small">{ELEMENT_INFO[e].good}</p></div>)}</div><div className="card judgeCard"><h2>判定</h2><div className="judge">{chart.strength.label}</div><p>{chart.strength.explanation}</p><div className="box"><small>喜神目安</small><b>{chart.strength.useful.join("・")}</b></div><div className="box"><small>補いたい五行</small><b>{chart.weakest}</b><p>{ELEMENT_INFO[chart.weakest].lucky}</p></div><div className="box dark"><small>空亡</small><b>{chart.voids.join("・")}</b></div></div></section>

        <section className="readings">{texts.map(([title, icon, text]) => <div className="card" key={title}><h3>{icon} {title}</h3><p>{text}</p></div>)}</section>

        <section className="card glossary"><h2>用語説明</h2><div className="gloss"><p><b>年柱</b>：家系、幼少期、外から見た印象。</p><p><b>月柱</b>：仕事、才能、社会運。四柱推命ではかなり重要。</p><p><b>日柱</b>：本人の本質、恋愛、結婚、自分らしさ。</p><p><b>時柱</b>：晩年、子ども、未来の可能性。</p><p><b>通変星</b>：才能や行動パターンを見る星。</p><p><b>十二運</b>：エネルギーの強さや出方を見るもの。</p><p><b>空亡</b>：こだわりすぎると空回りしやすいテーマ。</p><p><b>喜神</b>：運を整えるために使いやすい五行。</p></div></section>

        <section className="card conclusion"><h2>鑑定まとめ</h2><p>{name || "お客様"}様は、日主「{chart.dayMaster.k}」の性質を中心に、強い五行「{chart.strongest}」を才能として使うことで運が開きます。命式は「{chart.strength.label}」寄りのため、喜神目安である「{chart.strength.useful.join("・")}」を生活・仕事・人間関係に取り入れることが開運の鍵です。</p></section>
      </main>
    </div>
  </div>;
}

const css = `
*{box-sizing:border-box}body{margin:0;background:#f8fafc;color:#0f172a;font-family:-apple-system,BlinkMacSystemFont,'Hiragino Sans','Yu Gothic',Meiryo,sans-serif}.app{min-height:100vh;padding:22px;background:radial-gradient(circle at top left,#fff7ed,transparent 35%),linear-gradient(135deg,#fff,#f8fafc)}.hero{max-width:1180px;margin:0 auto 18px;padding:28px;border:1px solid #fde68a;border-radius:32px;background:rgba(255,255,255,.92);display:flex;justify-content:space-between;gap:20px;box-shadow:0 14px 40px rgba(15,23,42,.06)}.badge{display:inline-block;background:#0f172a;color:#fff;border-radius:999px;padding:9px 14px;font-weight:900}.hero h1{font-size:44px;margin:14px 0 8px;letter-spacing:-.04em}.hero p{color:#64748b;line-height:1.8;margin:0}.actions{display:flex;gap:10px;align-items:flex-end;flex-wrap:wrap}.btn{border:0;border-radius:18px;background:#0f172a;color:white;padding:13px 16px;font-weight:900;cursor:pointer}.btn.sub{background:white;color:#0f172a;border:1px solid #e2e8f0}.helpGrid{max-width:1180px;margin:0 auto 18px;display:grid;grid-template-columns:repeat(4,1fr);gap:14px}.help h3{margin:0 0 8px}.help p{margin:0;color:#475569;line-height:1.7}.layout{max-width:1180px;margin:auto;display:grid;grid-template-columns:330px 1fr;gap:22px}.card{background:rgba(255,255,255,.94);border:1px solid #fde68a;border-radius:28px;padding:22px;box-shadow:0 10px 32px rgba(15,23,42,.05)}.form{height:max-content;position:sticky;top:18px}.form h2,.card h2{margin:0 0 16px}.form label{display:block;margin:14px 0 7px;font-size:13px;font-weight:900}.form input,.form select{width:100%;border:1px solid #e2e8f0;border-radius:16px;padding:13px;font-size:16px;background:white}.note{margin-top:16px;border-radius:20px;padding:14px;line-height:1.7;font-size:13px;background:#fff7ed}.main{display:flex;flex-direction:column;gap:18px}.summary{display:flex;justify-content:space-between;align-items:center;gap:18px}.summary h2{font-size:32px;margin:4px 0}.summary p{color:#64748b;margin:5px 0}.small{font-size:12px}.muted{color:#64748b}.gold{color:#b45309}.master{background:#0f172a;color:white;border-radius:26px;padding:18px 28px;text-align:center;min-width:130px}.master span,.master em{display:block;opacity:.75;font-style:normal}.master b{font-size:46px;line-height:1}.explain p{line-height:1.8;color:#334155}.pillars{display:grid;grid-template-columns:repeat(4,1fr);gap:14px}.topline{display:flex;justify-content:space-between;gap:10px;align-items:center;color:#b45309}.topline span{background:#0f172a;color:white;border-radius:999px;padding:4px 9px;font-size:11px;font-weight:900}.bigKanji{font-size:38px;font-weight:900;letter-spacing:.08em;margin:10px 0}.mini{display:grid;grid-template-columns:1fr 1fr;gap:8px}.mini div{background:#f8fafc;border-radius:16px;padding:10px}.mini small,.box small{display:block;color:#64748b;font-size:11px}.mini b,.box b{display:block;margin-top:3px}.two{display:grid;grid-template-columns:1fr 330px;gap:18px}.barbox{margin:15px 0}.row{display:flex;justify-content:space-between;gap:10px;align-items:center}.bar{height:12px;background:#e2e8f0;border-radius:999px;overflow:hidden;margin:8px 0}.bar div{height:100%;background:#0f172a;border-radius:999px}.judge{font-size:38px;font-weight:900;margin:8px 0 10px}.judgeCard p{line-height:1.7;color:#475569}.box{background:#fff7ed;border-radius:20px;padding:15px;margin:10px 0}.box.dark{background:#0f172a;color:#fff}.box p{margin:6px 0 0;font-size:13px}.readings{display:grid;grid-template-columns:repeat(3,1fr);gap:14px}.readings h3{margin-top:0}.readings p,.conclusion p{line-height:1.9;color:#334155}.gloss{display:grid;grid-template-columns:repeat(2,1fr);gap:8px 18px}.gloss p{margin:0;line-height:1.7;color:#334155}@media(max-width:900px){.app{padding:14px}.hero,.summary{display:block}.layout,.helpGrid{grid-template-columns:1fr}.form{position:static}.pillars,.readings,.two,.gloss{grid-template-columns:1fr}.hero h1{font-size:34px}.master{margin-top:16px}}@media print{.noPrint{display:none!important}.app{padding:0;background:white}.hero,.card{box-shadow:none}.layout{display:block}.pillars{grid-template-columns:repeat(4,1fr)}.readings{grid-template-columns:repeat(2,1fr)}body{background:white}}
`;
