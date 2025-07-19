# React Native アプリの開発ガイドライン

本ファイルは、React Native + Expo で開発する場合のコーディングガイドラインです。コーディングを行う場合は、このガイドラインを遵守してください。

## 基本原則

### 1. 開発言語とフレームワーク

- React Native 単体ではなく、フレームワーク Expo を利用する
- **Expo SDK 53** (New Architecture有効)
- **Expo Router 5** (ファイルベースルーティング)
- **パッケージマネージャー**: Yarn 4.9.2

### 2. コード品質

- リンター/フォーマッター
    - リンターを 
- TypeScript
  - 厳格な型チェック必須
- コメントを書く
    - 自明な処理を除き、原則コメントを書く
    - JSDoc で、関数の目的や意図、引数、戻り値で記載する


## アーキテクチャパターン

### 1. Container Component パターン

全てのスクリーンは以下のパターンに従う：

```typescript
// プレゼンテーション コンポーネント
const ScreenComponent: React.FC<Props> = ({ ... }) => {
  const styles = useStyles()
  // UI レンダリングのみ
}

// ロジック コンテナ
const ScreenContainer: React.FC<Props> = (props) => {
  // 状態管理とビジネスロジック
  return <ScreenComponent {...props} {...logic} />
}

// 動的スタイル
const useStyles = makeStyles(() => useColorScheme(), (colorScheme) => {
  return StyleSheet.create({
    // COLOR(colorScheme)を使用したテーマ対応スタイル
  })
})
```

### 2. ディレクトリ構造

```root
src/
├── app/              # Expo Router ページ（ファイルベースルーティング）
│   ├── _layout.tsx   # スタックナビゲーション（ダークモード対応）
│   ├── index.tsx     # メイン画面（履歴表示）
│   ├── input.tsx     # 入力画面（音声入力対応）
│   ├── result.tsx    # 結果画面（音声出力対応）
│   └── settings.tsx  # 設定画面（データ管理）
├── components/       # 再利用可能UIコンポーネント
├── constants/        # 定数（Colors.ts）
├── hooks/            # カスタムReactフック
├── types/            # TypeScript型定義
└── utils/            # ユーティリティ関数
```

## スタイリング規約

### 1. ダークモード対応

```typescript
// Colors.ts から COLOR 関数を使用
import { COLOR } from '@/constants/Colors'

const styles = StyleSheet.create({
  container: styleType<ViewStyle>({
    backgroundColor: COLOR(colorScheme).BACKGROUND.PRIMARY,
    borderColor: COLOR(colorScheme).BORDER.PRIMARY,
  }),
  text: styleType<TextStyle>({
    color: COLOR(colorScheme).TEXT.PRIMARY,
  }),
})
```

### 2. 型安全なスタイル

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

### 3. 動的スタイル

```typescript
import { makeStyles } from 'react-native-swag-styles'

const useStyles = makeStyles(() => useColorScheme(), (colorScheme) => {
  return StyleSheet.create({
    // colorScheme に基づく動的スタイル
  })
})
```

## コンポーネント開発規約

### 1. 型定義

```typescript
// Props の型定義は明確に
type ComponentProps = {
  title: string
  onPress?: () => void
  // オプショナルプロパティには ? を使用
}

// Container Component 用の型継承
type Props = ComponentProps & {}
```

### 2. React Hooks の使用

```typescript
// useCallback を適切に使用してパフォーマンスを最適化
const handlePress = useCallback(() => {
  // イベントハンドラー
}, [依存配列])

// useFocusEffect でスクリーン同期
useFocusEffect(
  useCallback(() => {
    // フォーカス時の処理
  }, [依存配列])
)
```

### 3. エラーハンドリング

```typescript
try {
  // 非同期処理
} catch (error) {
  console.error('エラーログ:', error)
  Alert.alert('エラー', '日本語のユーザーフレンドリーなメッセージ')
}
```

## データ管理規約

### 1. AsyncStorage 使用

```typescript
// storage.ts のユーティリティ関数を使用
import { loadHistory, addHistoryItem, removeHistoryItem } from '@/utils/storage'

// 直接 AsyncStorage を使用せず、ユーティリティ関数を経由
```

### 2. 型定義

```typescript
// types/history.ts
export interface HistoryItem {
  id: string
  question: string
  answer: string
  timestamp: number
}
```

### 3. 状態管理

```typescript
// useState + useEffect の組み合わせ
const [data, setData] = useState<DataType[]>([])
const [isLoading, setIsLoading] = useState(true)

// 非同期データ読み込み
useEffect(() => {
  loadData()
}, [])
```

## API 統合規約

### 1. OpenAI API 統合

```typescript
// hooks/useAssistant.ts パターンを使用
const { status, response, fetchResponse } = useAssistant()

// ステータス管理: 'idle' | 'loading' | 'success' | 'error'
```

### 2. MCP サーバー統合

```typescript
// OpenAI Responses API + MCP tools の使用
const result = await client.responses.create({
  model: 'gpt-4.1',
  input: input,
  tools: [
    {
      type: 'mcp',
      server_label: 'yumemi-openhandbook',
      server_url: 'https://openhandbook.mcp.yumemi.jp/sse',
      require_approval: 'never',
    },
  ],
})
```

## ナビゲーション規約

### 1. Expo Router 使用

```typescript
import { useRouter } from 'expo-router'

const router = useRouter()

// フォワードナビゲーション
router.push('/screen-name')
router.push({ pathname: '/result', params: { data } })

// バックナビゲーション
router.back()
router.replace('/screen-name')
```

### 2. スクリーン間データ同期

```typescript
// useFocusEffect でスクリーン更新
useFocusEffect(
  useCallback(() => {
    refreshData()
  }, [refreshData])
)
```

## パフォーマンス最適化

### 1. FlashList 使用

```typescript
import { FlashList } from '@shopify/flash-list'

// 高性能リストレンダリング
<FlashList
  data={data}
  renderItem={renderItem}
  keyExtractor={keyExtractor}
  estimatedItemSize={80}
  showsVerticalScrollIndicator={false}
/>
```

### 2. useCallback の活用

```typescript
// レンダリング最適化
const renderItem = useCallback(
  ({ item }) => <ItemComponent item={item} />,
  [依存配列]
)

const keyExtractor = useCallback((item) => item.id, [])
```

## 国際化・ローカライゼーション

### 1. 日本語 UI

- 全てのユーザー向けテキストは日本語
- エラーメッセージも日本語で表示
- 日付フォーマット: `toLocaleString('ja-JP')`

### 2. アクセシビリティ

```typescript
// accessibilityRole と accessibilityLabel を適切に設定
<Pressable
  accessibilityRole="button"
  accessibilityLabel="ボタンの説明"
>
```

## 開発・テスト規約

### 1. 開発コマンド

```bash
# 依存関係インストール
yarn install

# 開発サーバー起動
npx expo start

# コード品質チェック
npm run lint
npx @biomejs/biome check --write ./src
```

### 2. 環境設定

```bash
# .env.local に必要な環境変数を設定
OPENAI_API_KEY=sk-...
```

### 3. プラットフォーム対応

- **iOS**: タブレット対応、New Architecture、ぼかし効果
- **Android**: エッジツーエッジUI、アダプティブアイコン
- **Web**: Metro バンドラー、静的出力

## 重要な実装原則

### 1. セキュリティ

- APIキーやシークレットを絶対にコミットしない
- ログに機密情報を出力しない
- セキュリティベストプラクティスに従う

### 2. エラーハンドリング

- catch ブロックでは日本語のユーザーフレンドリーなメッセージを提供
- 技術的な詳細をユーザーに公開しない
- デバッグ用にコンソールにエラーをログ出力

### 3. コードの一貫性

- 既存のコードパターンに従う
- 新しいライブラリを使用する前に、既存のコードベースで使用されているかチェック
- 命名規則とコーディングスタイルを統一

### 4. パフォーマンス

- 不要な再レンダリングを避ける
- 適切なメモ化を行う
- 非同期処理を適切にハンドル

このガイドラインに従うことで、保守性が高く、パフォーマンスに優れた、一貫性のあるコードベースを維持できます。
