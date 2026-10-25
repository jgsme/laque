# laque

jgs.me の design token。Tailwind CSS v4 の `@theme static` で書いた CSS 1 枚。

```sh
pnpm add laque
```

```css
@import "tailwindcss";
@import "laque/theme.css";
```

tailwindcss v4 は使う側で入れる (peerDependencies)。

## リリース

`package.json` の `version` を上げて commit し、同じ値の tag (`v0.1.1` など) を push すると、GitHub Actions が npm の staged publishing に上げる。npmjs.com の Staged Packages タブで Approve するか、`npm stage approve <stage-id>` を打つと公開される (どちらも 2FA が要る)。CI の認証は trusted publishing で、token は使わない。
