---
title: "nuxt v2系 + composition apiでナビゲーションガードをする方法"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxt", "vue", "typescript", "javascript", "compositionapi"]
published: true
---

# はじめに

composition-apiが出て、2系のNuxtとcomposition-apiを合わせて使う方が多いのではないでしょうか？

Nuxt 2系とcomposition-apiの組み合わせでなビゲーションガードを実装するのに難航したので忘備録として記事にしておきます。
（Nuxt 3系がリリースされたら多分使わなくなると思う）

version
```
Nuxt 2.15.xx
@nuxtjs/composition-api 0.26.0
```

# ナビゲーションガードをする

こちらの記事にあるように、
https://next.router.vuejs.org/guide/advanced/composition-api.html#navigation-guards

Vue3からは
```
import { defineComponent } from 'vue'
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

export default defineComponent({
  setup() {
    onBeforeRouteLeave((_to, _from, next) => {
      next()
    })

    onBeforeRouteUpdate((_to, _from, next) => {
      next()
    })
  },
})
```

のように`onBeforeRouteLeave, onBeforeRouteUpdate`を用いて書くことができますが、
これはVue2 (Nuxt2) + composition-api だと機能しません。

## コンポーネント内ガード

```
import { defineComponent } from '@nuxtjs/composition-api'

export default defineComponent({
  beforeRouteUpdate(_to, _from, next) {
    this.hoge() // ほげ〜
    next()
  },
  beforeRouteLeave(_to, _from, next) {
    this.hoge() // ほげ〜
    next()
  },
  setup() {
    const hoge = () => {
      // ほげ〜
    }
    return {
      hoge,
    }
  },
})
```

defineComponentの引数は
beforeRouteUpdateやbeforeRouteLeave,その他Vue2で使えるライフサイクルを記述することができるので、ここに書くことが可能です。
なお、setup()のreturnで返した関数に`this.`でアクセスすることもできます。

※なお。こちらはページコンポーネントしか動きません。

## グローバルガード

`src/plugins/globalSetup.ts`
グローバルセットアップ内で、`router.beforeResolve`を用いて実現可能です。
（他の場所に記述しても動くと思いますが、ここに書くのがいいかなと思っています）

```
import { onGlobalSetup, useRouter } from '@nuxtjs/composition-api'

export default () => {
  onGlobalSetup(() => {
    const router = useRouter()

    router.beforeResolve((_to, _from, next) => {
      next()
    })
  })
}
```

※なお、こちらは
`nuxt.config.ts`内で
```
plugins: ['plugins/globalSetup'],
```
する必要があります。ファイル名はなんでも大丈夫です。