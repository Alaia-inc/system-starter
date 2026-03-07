# プロジェクトの説明

## 概要

私たちのプロダクトでは独自の構成を使用しています。しかし基本は関数型プログラミングの原則、型の安全性に考慮した TypeScript のベストプラクティスに従います。

## 基本方針

- ヒアドキュメントは止めましょう。
- コメントは単純なものではなく文脈や実装意図について日本語で書きましょう。。悪い例：hooks にまとめる。良い例：製品化の際には端末本体に QR コードを貼り付けるため、ーーーする。
- まずはテストを書いてから実装する TDD のアプローチをとる。
- フロントエンドでテストは書かないでいいです。新しく vitest や jest をフロントエンドのパッケージに使用してはいけません。
- プロジェクトはスキーマ駆動開発を採用している。
- コード変更後の検証は `verify` skill を使い、変更対象に応じた必須コマンドを省略せず実行しましょう。
- 思考は英語で、回答は日本語でお願いします。
- 可読性と保守性を最優先とする
- 型安全性を確保する
- 一貫したコーディングスタイルを維持する
- Pythonを利用して出力やコード実装、コマンドを打たないでください。EOFも利用しないでください。
- container/presentationパターンを採用しましょう。
- 引き算でコードは実装すべきです。例えばスタイルを整えるのにcssを増やすのではなく減らす方法をまずは考えるのが必須です。
- 最小差分に拘らず、プロジェクトが大きくなっても保守が容易になることを優先し、サボらず実装するべきです。
- ユースケース層で権限を再チェックしない。
- 相対パスではなく絶対パスでimportしましょう。
- **定数の集約**: 要件/仕様に紐づく数値・文字列（例: 件数上限、サンプリング間隔、APIの固定パスなど）は `backend/src/constants` / `frontend/src/constants` に集約し、実装ファイル内にマジックナンバー/文字列を散在させない（アルゴリズム内部の局所値は除く）。
- **取得上限（take/limit）の扱い**: 正確性/整合性に影響するデータ取得の上限（Prisma の `take` / SQL の `LIMIT` など）を実装者判断で入れない。必要な場合は事前に相談し、仕様として合意した値を `backend/src/constants` / `frontend/src/constants` に定数化する。合意済みの上限を使う場合でも「上限を超えると誤動作する」設計は避け、ページング/探索などで網羅性と一貫性を担保する。
- このプロジェクトのルールに反するコードがあれば必ずフィードバックし、解決策を提案したり判断を仰いだりしましょう。
- N+1クエリのコードを書かないようにしましょう。
- **DBアクセスはPrisma Clientを優先**: `prisma.<model>` を用いた Prisma Client を原則とし、`$queryRaw` / `$executeRaw` による SQL 直書きは原則禁止（どうしても必要な場合は「Prisma Client で表現できない理由」を日本語コメントで明記し、事前に相談/承認を取る）。
- **ルート直DBアクセス禁止**: `backend/src/api/routes` では `prisma.<model>` を直接呼ばず、ユースケース層を経由する。DBアクセスは `backend/src/features/**/adapters/prisma*.ts` 等のアダプタに閉じ込める。
- **DBレコードの型はPrisma由来に寄せる**: `api/routes` や Prisma アダプタ等で「Prisma取得結果→DTO変換」のために内部型が必要な場合、`type X = { ... }` のように行型を手書きせず、`@prisma/client` の `Prisma.<Model>GetPayload` 等から導出してスキーマ変更に追従させる（API契約は `schemas/dto` を唯一の起点とする）。
- **Prismaの`select`/`include`方針**: 取得は原則 `select` で必要最小限の列だけを明示し、`include` は関連データが本当に必要な場合に限定する。`include` の安易な多用や `include: true` の連鎖による過剰取得を禁止する。
- **取得形状の責務を明示する**: 取得クエリは「どのユースケースでどの項目が必要か」を先に固定し、その形状に合わせて `select` / `include` を設計する。取得後に不要項目を捨てる前提の実装を禁止する。
- ロールに関する条件分岐では三項演算子を使わず、将来的な役割追加を見据えて必ずifまたはswitchで分岐を明示する。

## MCP

- OpenAI の API / Responses API / Agents SDK / Codex など OpenAI 製品に関する調査では、まず `openaiDeveloperDocs` MCP を使って公式ドキュメントを参照する。

## TypeScript コーディングルール

- **let 変数は使用禁止**: 常に`const`を使用し、再代入が必要な処理は避ける。
- **any 型の使用禁止**: 適切な型定義を常に行う。
- **型アサーションの回避**: `as`による型アサーションは避け、型ガードと早期リターンを使用。
- **非null断言の禁止**: `value!` のような非null断言で型エラーを握りつぶさない。必要な場合は境界で `parse` / `safeParse` し、処理分岐では型ガードで明示する。
- **明示的な戻り値型**: すべての関数に戻り値の型アノテーションを付ける。
- **型エイリアスの優先**: インターフェイスよりも型エイリアスを優先して使用。
- **ドキュメント化**: すべての公開関数には TSDoc コメントを付与。
- **検証の単一手順**: 変更後の検証は `verify` skill を唯一の起点とし、変更対象に応じた `lint` / `typecheck` / `test:coverage` を完走する。失敗や未実行を黙って省略してはいけない。
- エラーを隠蔽するフォールバックは絶対に書いてはいけません。例えば、チーム内のAを変更しようとした時に、チームが見つからない場合にデフォルトでチームを選択するなどもエラーの隠蔽です。デフォルト値を使うのは危険であることを理解しましょう。
- **監視通知の必須運用**: `catch` で処理した失敗でも、`5xx` 相当（運用で検知すべき失敗）は必ず Sentry へ送信する。`console.error` のみ・ログのみで終了する実装は禁止。`4xx` の業務エラーを意図的に返すケースは対象外だが、判定不能な失敗を `4xx` に寄せて隠蔽してはいけない。
- 不要なnullやundefined,分岐は増やさないべきです。認知的複雑度を意識しましょう。
- **Optionalの過剰拡張を禁止**: 必須として扱える値を `| null` / `| undefined` に広げ、後段の `normalize*` 関数で戻す実装を禁止する。値の検証・正規化は境界層で一度だけ行い、以降の層では最小の型で受け渡す。
- **共有DTOの union を狭めない**: `PermissionRequestDto['requests'][number]['permissionId']` のような union 型を、途中の `normalize*` で片側の enum（例: device 専用）に変換してはならない。正規化関数は入力/出力ともに DTO 契約と同じ型幅を維持する。
- **広すぎる入力型を禁止**: 契約が決まっている値に `| string` / `| unknown` を足して受け入れ範囲を広げる実装を禁止する。必要なら境界層で DTO schema による `parse`/`safeParse` を行い、内部は契約どおりの型だけを受け渡す。
- **enum/union を string に広げない**: Prisma/DTO 由来で enum/union として確定している値（例: `role`, `workspaceRole`, `status`, `permissionId`）を `string` で受け直すことを禁止する。必要な場合は `Prisma.<Model>GetPayload` や DTO 型から直接導出する。
- **safeParse失敗の握りつぶし禁止**: `safeParse` の失敗時にデフォルト値へフォールバック（例: `member` に寄せる）したり権限を暗黙変更する実装を禁止する。失敗は明示的にエラーとして扱い、必要な通知（Sentry）を行う。
- **不正値のサイレント除外禁止**: 配列やJSONの検証で未知値を黙って捨てる実装を禁止する。拒否するか、失敗として扱うかを仕様として明示し、実装に反映する。
- **想定内エラーは Result Union に統一**: 業務ルールで想定される失敗（not_found / conflict / invalid_request など）は `throw/catch` ではなく、判別可能な Result Union（discriminated union）で返す。`{ ok: true, data: ... } | { ok: false, reason: ... }` のように型で分岐を強制する。
- **Error.message 文字列比較の禁止**: `if (error.message === "...")` や `error.message.includes(...)` による業務分岐を禁止する。判定は Result Union の `reason`、または専用エラー型（`instanceof`）で行う。
- **例外の責務を限定する**: `throw` は予期しない障害（5xx 相当）に限定し、`catch` した障害は必ず `reportHandledError` などで監視通知する。外部ライブラリが型付きエラーを提供しない場合のみ、境界で専用エラー型へ変換してから扱う。
- **送信payloadもDTOで検証する**: フロントエンドの API 呼び出しでは、レスポンスだけでなくリクエストも `@alaia_gps/backend/schemas/dto/*` の schema で送信直前に検証し、契約外データを `typecheck` 任せにしない。
- **欠損表現の統一**: 同一責務の中で `null` と `undefined` を併用しない。DB保存値など仕様上 `null` が必要な箇所以外は `undefined` を使い、`null` を採用する場合は理由を日本語コメントで明記する。
- **欠損判定は明示比較を優先**: `undefined` を判定したい箇所で `if (value)` の truthy 判定を使わず、`if (value === undefined)` / `if (value !== undefined)` を使って意図を固定する。
- **型チェック抑制コメント禁止**: `@ts-ignore` / `@ts-expect-error` で型エラーを抑制する実装を禁止する（自動生成ファイルは除く）。抑制が必要に見える場合は型定義・境界検証・設計を修正する。

### スキーマ運用ルール

- `backend/src/generated/zod` は Prisma 由来の生スキーマ。フィールド構成の参照や `pick/extend` の起点として使うが、ここで `schema.parse` は行わない。
- `backend/src/schemas` は外部との契約・バリデーション用。HTTP 入口や Prisma アダプタなど境界層では必ずここにあるスキーマで `parse`/`safeParse` を実行し、実行時検証を行う。
- 境界層の例: `backend/src/routers/**/*`, `backend/src/server.ts`, `backend/src/features/**/adapters/prisma*.ts`, 外部 API 連携のアダプタ。
- ドメインやポート層では、境界層で検証済みのデータを扱う前提とし、`pick` や `extend` を用いて型 (`TypeOf`) を導出する。新規に `z.object` を定義するのは `schemas/` 配下に限定する。
- `schemas/` でスキーマを定義する際は、原則として `generated/zod` のスキーマから `pick` / `omit` / `extend` で導出する。どうしても `z.object` などで新規定義を行う場合は、理由を日本語コメントで明記する。
- `schemas/dto`で定義する場合はres/reqに合わせてapiとして利用できるように作る
- 手動でマイグレーションファイルやディレクトリは作成しない。
- **フロントエンドを含む境界レイヤでは DTO を唯一の契約ソースとする。** 型やスキーマを参照する際は `@alaia_gps/backend/schemas/dto/*` を経由し,
  DTO から離れると契約変化を見落とすため、境界実装では常に DTO を起点にする。

## Coding Conventions

- 全てのカスタムコンポーネントのファイル名はパスカルケースで記述します。(e.g., `UserProfile.tsx`)
- Utility functions should be in `utils/` directory with camelCase names
- 状態管理を使用するカスタム hooks は`frontend/src/hooks`ディレクトリの下に配置します。

## React / Hooks（アンチパターン禁止）

「外部システムとの同期」ではなく、**props/state から導出できる値のために `useEffect` で `setState` するのは禁止**です。
（不要な再レンダー・状態の二重管理・同期ずれの温床になります）

### 禁止（アンチパターン）

- props/state を「コピー」するための `useEffect`
  - 例: `useEffect(() => setValue(props.value), [props.value])`
- props/state から導出できる値を、別の `useState` に持つ（冗長 state）
  - 例: `const [fullName, setFullName] = useState('')` を持って `firstName/lastName` から更新する
  - 例: `visibleTodos` のような「フィルタ結果」を state + `useEffect` で保持する
- props の変更に合わせて state をリセット/調整する `useEffect`
  - 例: `useEffect(() => setError(null), [deviceId])`
- ユーザ操作起因の処理（API呼び出し/Toast表示/遷移など）を、フラグ state + `useEffect` で実行する
  - 例: `setPayload(...)` → `useEffect(() => post(payload), [payload])`
- state を「数珠つなぎ」に調整する `useEffect`（Effect chain）
- 入力値の正規化（trim/number化/format）を `useEffect` で行う
  - 例: `useEffect(() => setValue(value.trim()), [value])`（入力中のカーソル位置や編集体験を壊しやすい）
- データ取得を `useEffect` + `useState` で手書きする（このプロジェクトでは React Query を使う）
  - 例: `useEffect(() => { fetch(...).then(setData) }, [])`
- `useEffect` の依存配列をごまかす（意図的に deps を抜く / eslint を無効化する）
  - 例: `// eslint-disable-next-line react-hooks/exhaustive-deps`
- レンダー中に `setState` して状態を「調整」する（無限ループや挙動の理解が難しいため原則禁止）
  - 例: `if (prop !== prev) { setPrev(prop); setValue(...) }`

### どう書くか（推奨パターン例）

- **導出できる値はレンダー中に計算する（派生 state を持たない）**

```tsx
const [draft, setDraft] = useState<string | null>(null);
const value = draft ?? String(locationIntervalSeconds);
```

- **重たい計算だけ `useMemo` でキャッシュする（state + `useEffect` で持たない）**

```tsx
const visibleTodos = useMemo(
  () => getFilteredTodos(todos, filter),
  [todos, filter],
);
```

- **入力フォームは「ドラフト」だけ state に持ち、元データは props を信頼する（必要なら ID で紐付ける）**

```tsx
type Draft = { readonly deviceId: string; readonly value: string };
const [draft, setDraft] = useState<Draft | null>(null);
const value =
  draft?.deviceId === deviceId ? draft.value : String(locationIntervalSeconds);
```

- **端末/ユーザなど“対象が変わったら state を丸ごとリセットしたい”場合は `key` を使う**

```tsx
return <DeviceSettingsForm key={deviceId} deviceId={deviceId} />;
```

- **ユーザ操作起因の副作用はイベントハンドラ（または mutation の `onSuccess`）に閉じ込める**

```tsx
const onSubmit = (): void => {
  mutation.mutate(input, {
    onSuccess: () => toast({ title: "更新しました" }),
  });
};
```

- `useEffect` を使うのは「外部システムとの同期」が必要なときだけ（例: DOM API / subscribe / timer / analytics / 画面表示中のデータフェッチ等）

### Chakra/Ark の Switch/Checkbox 実装ルール

- `Switch.Root` / `Checkbox.Root` を制御コンポーネントとして使う場合、**状態変更イベント（`onCheckedChange` など）を必ず結線**し、UI表示だけで終わらせない。
- `checked` を渡す実装では、イベント内で必ず状態更新または mutation 呼び出しまで到達させる。クリックしてもネットワークリクエストが発火しない実装を禁止する。
- `HiddenInput` / `Control` / `Thumb` / `Label` など、既存プリミティブの推奨構成を崩さない（独自DOMで置き換えてアクセシビリティを壊さない）。

## OpenAPI 構成と型共有

- **スキーマの単一ソース**: Fastify ルート定義は `backend/src/lib/openapi.ts` を経由して OpenAPI に登録し、`pnpm -C backend run generate:openapi` で `frontend/openapi.json` を生成します（生成物のため Git 管理しません）。生成後は `pnpm -C backend run verify:openapi` で OpenAPI スキーマ検証を行い、生成結果の正当性を確認します。
- **クライアントコードの自動生成**: `frontend/openapi.json` から `pnpm -C frontend run generate:openapi-client` を実行すると `frontend/src/lib/api-client/` に型付きクライアントが生成されます。手作業で編集すると再生成時に失われるため絶対に避けてください。
- **型共有の方針**: フロントエンド固有の型定義を作らず、`@alaia_gps/backend/schemas/<category>/<file>` から Zod スキーマと型エイリアスを取り込み、OpenAPI で取得したレスポンスを `schema.parse` で検証して整合性を保証します。端末に組み込む管理画面でも同じ定義を流用できるよう、この方式で一元管理します。
- **API 呼び出しのカプセル化**: 生成クライアントの `__request` を直接コンポーネントで呼ばず、`frontend/src/lib/api/*.ts` で明示的なユースケース関数を定義し、必要に応じて `frontend/src/hooks` 配下のカスタムフックで React Query などの状態管理と結合します。下記のようにフックへ閉じ込めてください。

```typescript
export const useWorkspaceList = () => {
  return useQuery({
    queryKey: ["workspaces"],
    queryFn: () => listWorkspaces(),
  });
};
```

## E2Eテストについて

- Screen Action Patternを採用
- PageObject: UI要素の定義のみ。ActionObject: 操作の定義のみ。StateObject: 状態検証のみ。ScreenObject: 上記3つを統合。ScenarioFile: テストケースの定義(ログイン->申請作成ボタン->申請作成)

## 環境変数について

- 本プロジェクトでは環境変数を各層のenvディレクトリで管理している。
- process.envをconfigファイル以外では利用してはいけない。
- 環境変数は必ずconfigファイルから取得し、　undefinedや存在チェックを各ファイルで行わないようにする
- 新しい環境変数を利用する場合は必ず各envディレクトリの`.env.<scope>.local`と`.env.<scope>.development`に記載しましょう。暗号化は私がするのであなたはしなくて良いです。

---

name: ui-skills
description: Opinionated constraints for building better interfaces with agents.

---

# UI Skills

When invoked, apply these opinionated constraints for building better interfaces.

## How to use

- `/ui-skills`
  Apply these constraints to any UI work in this conversation.

- `/ui-skills <file>`
  Review the file against all constraints below and output:
  - violations (quote the exact line/snippet)
  - why it matters (1 short sentence)
  - a concrete fix (code-level suggestion)

## Stack

- MUST use Tailwind CSS defaults unless custom values already exist or are explicitly requested
- MUST use `motion/react` (formerly `framer-motion`) when JavaScript animation is required
- SHOULD use `tw-animate-css` for entrance and micro-animations in Tailwind CSS
- MUST use `cn` utility (`clsx` + `tailwind-merge`) for class logic

## Components

- MUST use accessible component primitives for anything with keyboard or focus behavior (`Base UI`, `React Aria`, `Radix`)
- MUST use the project’s existing component primitives first
- NEVER mix primitive systems within the same interaction surface
- SHOULD prefer [`Base UI`](https://base-ui.com/react/components) for new primitives if compatible with the stack
- MUST add an `aria-label` to icon-only buttons
- NEVER rebuild keyboard or focus behavior by hand unless explicitly requested

## Interaction

- MUST use an `AlertDialog` for destructive or irreversible actions
- SHOULD use structural skeletons for loading states
- NEVER use `h-screen`, use `h-dvh`
- MUST respect `safe-area-inset` for fixed elements
- MUST show errors next to where the action happens
- NEVER block paste in `input` or `textarea` elements

## Animation

- NEVER add animation unless it is explicitly requested
- MUST animate only compositor props (`transform`, `opacity`)
- NEVER animate layout properties (`width`, `height`, `top`, `left`, `margin`, `padding`)
- SHOULD avoid animating paint properties (`background`, `color`) except for small, local UI (text, icons)
- SHOULD use `ease-out` on entrance
- NEVER exceed `200ms` for interaction feedback
- MUST pause looping animations when off-screen
- SHOULD respect `prefers-reduced-motion`
- NEVER introduce custom easing curves unless explicitly requested
- SHOULD avoid animating large images or full-screen surfaces

## Typography

- MUST use `text-balance` for headings and `text-pretty` for body/paragraphs
- MUST use `tabular-nums` for data
- SHOULD use `truncate` or `line-clamp` for dense UI
- NEVER modify `letter-spacing` (`tracking-*`) unless explicitly requested

## Layout

- MUST use a fixed `z-index` scale (no arbitrary `z-*`)
- SHOULD use `size-*` for square elements instead of `w-*` + `h-*`

## Performance

- NEVER animate large `blur()` or `backdrop-filter` surfaces
- NEVER apply `will-change` outside an active animation
- NEVER use `useEffect` for anything that can be expressed as render logic

## Design

- NEVER use gradients unless explicitly requested
- NEVER use purple or multicolor gradients
- NEVER use glow effects as primary affordances
- SHOULD use Tailwind CSS default shadow scale unless explicitly requested
- MUST give empty states one clear next action
- SHOULD limit accent color usage to one per view
- SHOULD use existing theme or Tailwind CSS color tokens before introducing new ones
