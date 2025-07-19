# アーキテクチャ

React Native アプリで利用するアーキテクチャを定めます。

## 基本方針

状態管理およびビジネスロジックの基本的なアーキテクチャとして **Redux （Redux Toolkit） と Redux Saga** を採用します。

- Redux はアプリケーション全体で共有される状態（State）を管理する。
- Redux Saga は API 通信やデータベースアクセスなどの副作用（Side Effect）を伴う非同期処理を管理する。

ただし、状態やロジックが特定の画面やコンポーネントに限定される場合は、その限りではありません。コンポーネントの再利用性や関心の分離を考慮し、よりシンプルな解決策を選択することを推奨します。

## 状態管理 （State Management）

### グローバルな状態管理

アプリケーション全体で必要とされる状態（例: ユーザー情報、認証状態、共通の設定など）は、Redux を使用して管理します。

- 状態の更新ロジックは [Redux Toolkit の `createSlice`](https://redux-toolkit.js.org/api/createSlice) を用いて記述する。
- 原則として、Container Component（画面コンポーネント）から `useSelector` を用いて State を参照する。

### ローカルな状態管理

特定の画面やコンポーネント内でのみ利用される状態（例: フォームの入力値、UI の開閉状態など）は、React 標準の `useState` や `useReducer` を用いて管理します。これにより、Redux ストアの肥大化を防ぎ、コンポーネントの独立性を高めます。

## 副作用・非同期処理 （Side Effects / Asynchronous Actions）

### グローバルな非同期処理

複数の画面で共通して利用される非同期処理や、アプリケーションのコアとなるビジネスロジックは Redux Saga を用いて実装します。

**原則:**

- API 通信など、アプリケーション全体に関わる副作用は Redux Saga で管理する。
- Saga は、特定のアクションを監視し、非同期処理を実行して、結果に応じて新しいアクションを `put` する。

### ローカルな非同期処理

特定の画面内でのみ完結する非同期処理（例: 特定画面専用のデータ取得）については、Redux Saga を使わず、コンポーネント内で完結させる方法も許容します。

**選択肢:**

- カスタムフック（Custom Hooks）は非同期処理と関連する状態をカプセル化する。これにより、ロジックの再利用が容易になり、コンポーネントの見通しがよくなる。
- シンプルな一度きりの API リクエストなど、複雑な状態管理を伴わない場合は、`useEffect` 内で直接的に非同期処理を実行しても構わない。

## Redux Saga の導入と実装例

### 1. 必要なライブラリのインストール

```bash
yarn add redux react-redux @reduxjs/toolkit redux-saga
```

### 2. Store の設定

Redux Toolkit と Redux Saga を連携させるためのストア設定です。

`src/store.ts`

```typescript
import { configureStore } from '@reduxjs/toolkit';
import createSagaMiddleware from 'redux-saga';
import rootReducer from './reducers'; // `combineReducers` でまとめたReducer
import rootSaga from './sagas'; // `all` でまとめたSaga

// Sagaミドルウェアを作成
const sagaMiddleware = createSagaMiddleware();

// Storeの設定
export const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(sagaMiddleware),
});

// Sagaを起動
sagaMiddleware.run(rootSaga);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 3. Saga の実装例

ユーザー情報を取得する Saga の例です。

`src/features/user/userSaga.ts`

```typescript
import { call, put, takeEvery } from 'redux-saga/effects';
import { fetchUserApi } from './api'; // APIリクエストを行う関数
import { userSlice } from './userSlice';

// userSlice.actionsからAction Creatorを取得
const { fetchUserSuccess, fetchUserFailure } = userSlice.actions;

// Worker Saga
function* handleFetchUser(action) {
  try {
    const user = yield call(fetchUserApi, action.payload);
    yield put(fetchUserSuccess(user));
  } catch (error) {
    yield put(fetchUserFailure(error.message));
  }
}

// Watcher Saga
export function* watchFetchUser() {
  // `user/fetchUser` アクションを監視し、実行されたら `handleFetchUser` を呼び出す
  yield takeEvery('user/fetchUser', handleFetchUser);
}
```

## カスタムフックの実装例

特定の画面でのみデータを取得する場合のカスタムフックの例です。

`src/hooks/useFetchArticle.ts`

```typescript
import { useState, useEffect } from 'react';
import { fetchArticleApi } from '../api/article'; // 特定のAPIリクエスト

interface Article {
  id: string;
  title: string;
  content: string;
}

export const useFetchArticle = (articleId: string) => {
  const [article, setArticle] = useState<Article | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchArticle = async () => {
      try {
        setLoading(true);
        const data = await fetchArticleApi(articleId);
        setArticle(data);
      } catch (err) {
        setError('記事の取得に失敗しました。');
      } finally {
        setLoading(false);
      }
    };

    fetchArticle();
  }, [articleId]);

  return { article, loading, error };
};
```
