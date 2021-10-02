---
title: "実務で学んだReactのデザインパターンとTips"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['react', 'typescript']
published: false
---

# はじめに
この記事では、ボク自身が開発しているプロジェクトで学んだ Reactのデザインパターン, Tipsの備忘録として、残させていただきます。
# props
1. props の初期値に値を指定する
   関数の引数に初期値を定義することができます。

```javascript
type Props = {
  color: string,
  text: string
}

const Button: React.VFC<Props> = ({ color = "red", text = 'text something here' }) => {
  return (
    <button style={color}>
        {text}
    </button>
  )
}
```
2.レスト構文
複数の props をひとまとまりにし、オブジェクトにします。
関数の引数や分割代入での多数の値を配列として受け取る場合に使用しますね。

```javascript
type ButtonProps = React.HTMLAttributes<HTMLButtonElement>

type Props = {
    color: string
} & ButtonProps;

const Button: React.FC<Props> = ({ color = "red", ...buttonProps }) => {
    return <button style={{ color }} {...buttonProps}>{children}</button>
}
```
3. props getters
   React Hooks以前に普及された Render Props と似ています。
   propsのオブジェクトを定義、一度だけ渡すというのもです。エンドユーザーは、ほしいprops やそうでないprops、追加したいprops を選択できるのはメリットかもしれないですね。

```javascript
import { useState } from "react";
function useButton () {
    const [isOepn, setisOpen] = useState(false)
    const toggle = () => setisOpen(!isOepn)

    const getTogglerProps = ({ onClick, ...otherProps } = {}) => ({
        'aria-expanded': isOpen,
        onClick: () => {
            onClick && onClick(isOpen)
            toggle(isOepn)
        },
        ...otherProps,
    })

    return {
        isOepn,
        getTogglerProps,
    };
}
```

```javascript
import { useButton} from "./useButton";

function Usage() {
  const {isOpen, getTogglerProps} = useButton();

  return (
    <>
      <button {...getTogglerProps()}>
          {isOpen ? "OPEN" : "CLOSE"}
      </button>
      <button {...getTogglerProps({ disabled: true })}>
      </button>
    </>
  );
}

export { Usage };
```

# さいごに
まだまだ学んだ事があるので、まとめていきたいなと思ってます。
少しでも役に立てたら嬉しいです。
