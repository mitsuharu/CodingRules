# コンポーネント



## カラーパターン

アプリで利用する色は個別では定義せずに、カラーデザインパターンを定義して利用する。

```typescript
import type { ColorSchemeName } from 'react-native'
type ColorPattern = {
    PRIMARY: string
    SECONDARY: string
    EMPHASIZE: string
  }

type ColorType = {
  CLEAR: string
  TEXT: ColorPattern
  BACKGROUND: ColorPattern
}

const defaultColor: ColorType = {
  CLEAR: 'rgba(0,0,0,0)',
  TEXT: {
    PRIMARY: 'black',
    SECONDARY: '#4F5A6B',
    EMPHASIZE: '#ffffff',
  },
  BACKGROUND: {
    PRIMARY: '#FFFFFF',
    SECONDARY: '#F2F2F2',
    EMPHASIZE: '#007AFF',
  },
}

const darkColor: ColorType = {
  ...defaultColor,
  TEXT: {
    PRIMARY: 'white',
    SECONDARY: '#E2E8F1',
    EMPHASIZE: '#ffffff',
  },
  BACKGROUND: {
    PRIMARY: '#171F2A',
    SECONDARY: '#11161D',
    EMPHASIZE: '#007AFF',
  },
}

export const COLOR = (colorScheme: ColorSchemeName = 'light') => {
  return colorScheme === 'dark' ? darkColor : defaultColor
}
```

## style

### 型安全

```typescript
import type { ImageStyle, TextStyle, ViewStyle } from 'react-native'

export const styleType = <T extends ViewStyle | TextStyle | ImageStyle>(
  style: T,
): T => style
```


```typescript
import { styleType } from '@/utils/styles'

// ViewStyle、TextStyle、ImageStyle の型安全性を確保
const styles = StyleSheet.create({
  container: styleType<ViewStyle>({
    flex: 1,
    // ...
  }),
  text: styleType<TextStyle>({
    fontSize: 16,
    // ...
  }),
})
```

### 動的スタイル

react-native-swag-styles https://github.com/ef-eng/react-native-swag-styles を利用する

```typescript
import { makeStyles } from 'react-native-swag-styles'

const Component: React.FC<ComponentProps> = ({ ... }) => {
  const styles = useStyles()
}

const useStyles = makeStyles(() => useColorScheme(), (colorScheme) => {
  return StyleSheet.create({
    // colorScheme に基づく動的スタイル
  })
})
```



## Container Component パターン

画面コンポーネントは、直接作らず、プレゼンテーションのComponent、ロジック管理のContainerと分けて定義する。

Componentは画面作成に専念する。
Containerは、Componentへの依存注入する形として、責務を分離する


```typescript
type ComponentProps = {}
type Props = ComponentProps & {}

// プレゼンテーション コンポーネント
const Component: React.FC<ComponentProps> = ({ ... }) => {
  const styles = useStyles()
  // UI レンダリングのみ
}

// ロジック コンテナ
const Container: React.FC<Props> = (props) => {
  // 状態管理とビジネスロジック
  return <ScreenComponent {...props} {...logic} />
}

// 動的スタイル
const useStyles = makeStyles(() => useColorScheme(), (colorScheme) => {
  return StyleSheet.create({
    // COLOR(colorScheme)を使用したテーマ対応スタイル
  })
})

export default Container
```
