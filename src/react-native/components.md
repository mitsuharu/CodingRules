# コンポーネント設計ガイドライン

React Native におけるコンポーネントの設計、作成、スタイリングに関するルールを定めます。一貫性、再利用性、保守性の高いコンポーネント開発を目指します。

## 基本方針

- Presentational and Container Components パターンを採用し、UI（見た目）とロジック（振る舞い）を分離してコンポーネントの責務を明確にする。
- TypeScript を活用し、Props や Style の型安全性を確保する。
- ライトモードとダークモードに対応可能なスタイリングを前提とする。

## ディレクトリ構成

コンポーネントは役割に応じて次のディレクトリに配置します。

- `src/components`: アプリケーション全体で再利用可能な汎用コンポーネント。
- `src/features/{feature_name}/components`: 特定の機能（画面）に特化し、再利用を想定しないコンポーネント。

## Presentational and Container Components パターン

コンポーネントは、UI を担当する **Presentational Component** と、ロジックを担当する **Container Component** に役割を分割します。これにより、UI とビジネスロジックの関心を分離します。ファイルは分割せず、同一ファイル内に記述し、Container をデフォルトエクスポートします。

### Presentational Component

- UI のレンダリングに専念する責務をもつ。
- 特徴
  - 状態を持たず、Props として受け取ったデータを表示する。
  - Redux ストアや Saga に直接アクセスしない。
  - ユーザー操作は、Props として受け取ったコールバック関数を呼び出すことで Container に伝える。
  - スタイリング（`StyleSheet`）はこのコンポーネントで定義する。

### Container Component

- ビジネスロジックと状態管理を担当する責務をもつ。
- 特徴
  - Redux ストアから状態を取得する (`useSelector`)。
  - アクションをディスパッチする (`useDispatch`)。
  - カスタムフックや Saga を利用して非同期処理を実行する。
  - Presentational Component に必要なデータとコールバック関数を Props として渡す。

### 実装例

`ProfileScreen.tsx` のように、1 つのファイル内に Component と Container を記述します。

```typescript
import React from 'react';
import { View, Text, Button, StyleSheet, type ViewStyle, type TextStyle } from 'react-native';
import { useSelector, useDispatch } from 'react-redux';
import { makeStyles, useColorScheme } from 'react-native-swag-styles';
import { COLOR } from '@/styles/color'; // カラーパレット
import { userSelector, fetchUser } from '@/features/user/userSlice';

// Presentational Componentが受け取るPropsの型定義
type ComponentProps = {
  userName: string;
  onPressReload: () => void;
};

// Container Componentが受け取るPropsの型定義 (画面遷移時など)
type Props = ComponentProps & {};

// Presentational Component
const Component: React.FC<ComponentProps> = ({ userName, onPressReload }) => {
  const styles = useStyles();

  return (
    <View style={styles.container}>
      <Text style={styles.text}>こんにちは, {userName} さん</Text>
      <Button title="更新" onPress={onPressReload} />
    </View>
  );
};

// Container Component
const Container: React.FC<Props> = (props) => {
  const dispatch = useDispatch();
  const user = useSelector(userSelector);

  const handleReload = () => {
    dispatch(fetchUser({ userId: '123' }));
  };

  return (
    <Component
      {...props}
      userName={user.name}
      onPressReload={handleReload}
    />
  );
};

// 動的スタイル
const useStyles = makeStyles(useColorScheme, (colorScheme) => {
    const styles = StyleSheet.create({
      container: styleType<ViewStyle>({
        flex: 1,
        backgroundColor: COLOR(colorScheme).BACKGROUND.PRIMARY,
        alignItems: 'center',
        justifyContent: 'center',
      }),
      text: styleType<TextStyle>({
        color: COLOR(colorScheme).TEXT.PRIMARY,
        fontSize: 18,
      }),
    });
    return styles;
  },
);

export default Container;
```

## スタイリング （Styling）

### 動的スタイルの実装

テーマ（ライト/ダークモード）の切り替えに対応するため、`react-native-swag-styles` を利用して動的にスタイルを生成します。

```typescript
import { makeStyles } from 'react-native-swag-styles';
import { useColorScheme, StyleSheet, type ViewStyle, type TextStyle  } from 'react-native';
import { COLOR } from '@/styles/color';

const useStyles = makeStyles(useColorScheme, (colorScheme) => {
    const styles =  StyleSheet.create({
      container: styleType<ViewStyle>({
        backgroundColor: COLOR(colorScheme).BACKGROUND.PRIMARY,
      }),
      text: styleType<TextStyle>({
        color: COLOR(colorScheme).TEXT.PRIMARY,
      }),
    });
    return styles;
  },
);
```

### カラーテーマ

アプリケーションで使用する色は、次のカラーパレットに集約します。コンポーネント内で直接色コードを指定することは禁止します。

`src/styles/color.ts`

```typescript
import type { ColorSchemeName } from 'react-native';

type ColorPattern = {
  PRIMARY: string;
  SECONDARY: string;
  EMPHASIZE: string;
};

type ColorType = {
  CLEAR: string;
  TEXT: ColorPattern;
  BACKGROUND: ColorPattern;
};

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
};

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
};

export const COLOR = (colorScheme: ColorSchemeName = 'light') => {
  return colorScheme === 'dark' ? darkColor : defaultColor;
};
```

### 型安全なスタイル（参考）

`react-native-swag-styles` を使用する場合、型推論が強力に機能するため必須ではない。ただし、`StyleSheet.create` を直接利用する際は、次のユーティリティで型安全性を担保できる。

`src/utils/styles.ts`

```typescript
import type { ImageStyle, TextStyle, ViewStyle } from 'react-native';

export const styleType = <T extends ViewStyle | TextStyle | ImageStyle>(
  style: T,
): T => style;
```
