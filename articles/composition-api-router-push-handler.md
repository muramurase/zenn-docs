---
title: "vue-router の router.pushがエラーを投げなくなった"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxt", "vue", "typescript", "javascript", "compositionapi", "vue-router"]
published: true
---

# はじめに

optional apiの時のようにrouter.push()をエラーハンドリングしようとしたら動かなくて焦った。
```
const router = useRouter()

try {
  router.push({ path: uri })
} catch (err) {
  console.log(err)
}
```

version
```
Nuxt 2.15.xx
@nuxtjs/composition-api 0.26.0
vue-router 3.5.1
```

# 事象

上記のように書いても何も起こらないので、  
下記のように記載すると...

```
const router = useRouter()

router
  .push({
    path: uri
  })
  .catch((err) => {
    console.log(err)
  })
```

undefinedが返っている...
```
TypeError: Cannot read properties of undefined (reading 'catch')
```

（router.d.tsを見てみると返り値の型定義がPromise<Route>なのでバグなのかもしれない）
```
push(location: RawLocation): Promise<Route>
```

# このように書く

上記がバグなのかどうかは分からないが、  
第２引数,第３引数にコールバック関数を渡せるようなのでそれを用いる。  

```
router.push(
  {
    path: uri
  },
  // 完了時（失敗時も動く）
  () => {},
  // 失敗時
  (err) => {
    console.log(err)
  }
)
```

なおerrの型は
```
interface Error {
  name: string;
  message: string;
  stack?: string;
}
```

router.replaceも同様。