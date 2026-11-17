# laque

このファイルは、この repo で作業するエージェント向けの情報をまとめる。

## 何の repo か

jgs.me の design token。npm の `laque` として公開している。中身は `src/theme.css` の 1 枚だけで、Tailwind CSS v4 の `@theme static` で書いてある。build も test も無い。

使っているのは次の 3 つ。

- `jgsme/jgs.me` の `packages/web` と `packages/img`
- `jgsme/sonde` の `packages/web`

値の一覧は design system「laque」(Claude の Design System の artifact) にもあるが、あちらは写し。正はこの repo の `src/theme.css`。値を変えたら design system も直す。

## 構成

| ファイル | 中身 |
| --- | --- |
| `src/theme.css` | token の全部。`package.json` の `exports` が `laque/theme.css` として出す |
| `package.json` | `version` を持つ。`tailwindcss` は peerDependencies (使う側が入れる) |
| `.github/workflows/release.yml` | `v*` の tag で npm の staged publishing に上げる |

`theme.css` の中は `/* --- 見出し --- */` で区切ってある。

| 区切り | 中身 |
| --- | --- |
| ブランド | `brand` `paper` |
| 文字と地 | `fg` `bg` `fg-muted` `fg-subtle` `fg-quote` `link`、引用の書体 `--font-quote` |
| 闇落ち | jgs.me の HTML 記事のヘッダが使う色 |
| 面と境界 | `surface` `surface-strong` `border-subtle` `border` `border-strong` `ink` |
| サイズ | 余白と寸法の語彙、コンテナ幅 |
| 文字サイズ | UI の 11 段と、本文の強調の 10 段 |
| 封じるパレット | gray / zinc / stone を消す |

値をこのファイルに写さない。`theme.css` のコメントが、なぜその値かの記録になっている。token を足す・変える時は、同じ調子でコメントに理由を書く。

## 語彙を封じる

この repo の役目は token を配ることと、語彙の外を書けなくすること。Tailwind の既定を `initial` で消してから、使ってよい値だけを宣言し直している。

| 封じているもの | 残している語彙 |
| --- | --- |
| `--spacing: initial` | 0 1 2 3 4 6 8 12 16 24 32 48 64 96 128 192 256 と `header` |
| `--container-*: initial` | `content` `quote` `photo` `rule` `video` `deco` |
| `--text-*: initial` | UI の `2xs xs sm base lg xl 2xl 3xl 4xl 5xl 6xl`、強調の `deco-1`〜`deco-10` と `deco-1-mobile`〜`deco-10-mobile` |
| `--color-gray-*` `--color-zinc-*` `--color-stone-*` | neutral (token の実体) と slate (jgs.me の on-this-day が使う) |

語彙の外を書くと class が生成されないだけで、build も test も通る。本番で余白や文字の大きさだけが消える。書いた時点で落とす役は jgs.me の `scripts/lint-tokens.sh` が持っている。語彙を足す・減らす時は、あちらの検査と文言も一緒に直す。

## 文字サイズ

UI と記事本文で体系を分けている。段を揃えない。

- UI の `xs`〜`6xl` は Tailwind 4.3.3 の既定と同じ値、同じ line-height。`2xs` (11px) だけ足してある
- 本文の強調 (`deco-N`) は Scrapbox の `[* ]`〜`[********** ]` に 1 段ずつ対応する。`text-deco-3` と書くだけで大きさ・行送り・太さ (700) が決まる
- `deco-N` は md (768px) 以上、`deco-N-mobile` は md 未満で使う。使う側は `text-deco-3-mobile md:text-deco-3` と並べて書く
- 刻みは等比ではなく、上の段ほど倍率が開く。md 以上は 18px〜144px、md 未満は 18px〜72px
- `--container-deco` (90vw) は、強調の行を本文幅の外へ広げる時の幅。値は `--container-photo` / `--container-rule` と同じだが、役割が別なので名前を分けている

## ハマりどころ

- **`initial` は宣言し直すより前に置く。** 後ろに置くと、直前に宣言した token まで消える。`--container-*` と `--text-*` がこれ
- **`--text-*: initial` は `--text-shadow-*` を消さない。** Tailwind 4.3.3 で build して確かめた。Tailwind を上げた時は下の「確かめ方」で `text-shadow-sm` が生成されるのをもう一度見る
- **class 名は使う側の source の文字列から拾われる。** `` `text-deco-${n}` `` のように組み立てた class は生成されない。使う側で完全な文字列として書く
- **`@theme static` なので、使われていない token も CSS 変数として全部出る。** 使う側が何を書くかに依存させないため。`@theme` に変えない
- **`--container-content` を変えたら jgs.me の `packages/web/utils/listImage.ts` と `bodyImage.ts` の `sizes` も直す。** CSS だけが変わって、画像の `sizes` は追随しない
- **色は hex を書き写さず Tailwind の変数を参照する。** v4 のパレットは OKLCH で、hex に写すと色がずれる

## 確かめ方

test は無い。token を変えたら、Tailwind で実際に build して生成される class を見る。

```sh
mkdir -p /tmp/laque-check && cd /tmp/laque-check
npm init -y >/dev/null && npm i tailwindcss@4.3.3 @tailwindcss/cli@4.3.3 >/dev/null
cat > in.css <<'CSS'
@import "tailwindcss";
@import "<この repo の絶対パス>/src/theme.css";
@source inline("text-deco-10 text-deco-10-mobile md:text-deco-3 text-2xs text-6xl text-7xl text-shadow-sm max-w-deco");
CSS
npx tailwindcss -i in.css -o out.css
grep -c '\.text-7xl' out.css     # 0 (語彙の外は生成されない)
grep -c '\.text-shadow-sm' out.css # 1 以上 (巻き添えで消えていない)
grep -A4 '\.text-deco-10 {' out.css # font-size / line-height / font-weight の 3 つ
```

`@source inline(...)` に、確かめたい class を並べる。

## リリース

1. branch で `package.json` の `version` を上げて PR を出し、merge する
2. main で同じ値の tag (`v0.3.0` など) を push する
3. GitHub Actions が npm の staged publishing に上げる。tag と `version` がずれていると落ちる
4. npmjs.com の Staged Packages タブで Approve するか、`npm stage approve <stage-id>` を打つ。どちらも 2FA が要る

CI の認証は trusted publishing (OIDC) で、npm の token は持たない。2FA 無しでは公開まで通らない作りにしてある。

公開したら、使う側 (`jgs.me` の `packages/web` など) の `laque` の version を上げる。

## 作業の決まり

- main に直接 commit しない。最新の `origin/main` から branch を切って PR を出す
- worktree は `.worktrees/<name>` に切る (`.gitignore` 済み)
- CLAUDE.md は作らない。作業に要る情報はこのファイルに書く
- README は短く保つ。何の repo かと最小の使い方だけ
