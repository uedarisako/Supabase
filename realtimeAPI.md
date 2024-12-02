# SupabaseとRealtime APIについて
## Supabaseとは
- 迅速にバックエンド機能を構築できるオープンソースのBaaS (Backend As A Service)プラットフォームであり、Firebaseの代替としても利用されている。
- クライアントから直接アクセスできるため、フロントエンドの開発者がバックエンドの知識なしで簡単に機能を追加できる。
- 主な機能には以下のものがある。
  - データベース
    - PostgreSQLを基盤にしたリレーショナルデータベースを提供している。これにより、強力で拡張性のあるデータ管理を簡単に行うことができ、SQLクエリを用いてデータ操作ができる。
  - 認証
    - ユーザー登録、ログイン、ログアウト、パスワードリセットなどの基本的な機能をサポートしている。
  - リアルタイム機能
    - データベースに対する変更（INSERT、UPDATE、DELETEなど）をリアルタイムでクライアントに通知できる。
  - ストレージ
    - Supabaseは、ファイルのアップロード、管理、ダウンロードができるストレージ機能を提供している。
  - Edge Functions
    - SupabaseのEdge Functionsは、サーバーレスで実行可能な関数を提供している。これにより、APIエンドポイントを作成したり、バックエンドの処理を行ったりできる。

### supabaseを使用するメリット
- 複数のテーブルを紐づけて利用することができるので、複雑なデータ構造を管理しやすい。
- アプリの仕様変更があっても、既存データを結合して対応しやすい

## Realtime APIとは 
- **Realtime API**は、Supabaseが提供するリアルタイムデータの同期機能。これを使用することで、データベースの変更をリアルタイムでクライアント（webブラウザなど）に通知することができる。
- 通常、クライアント（ブラウザなど）がサーバーにリクエストを送信し、サーバーの変更を反映させるが、Realtime APIを使うと、再リクエストなしでリアルタイムに変更を受け取ることができる。
- 例えば、あるユーザーがデータを追加した際に、他のユーザーや管理者がそのデータを即座に見ることができるようになる
- 実際の例としては、売り上げの即時把握やホテルなどの予約管理システムなど

### Realtime APIを使うメリット
- 即時性
    - ユーザーが行った操作がリアルタイムで反映される他のユーザーにも即座に通知される。これにより、迅速な情報共有が可能になる。
- シンプルな実装
    - Supabase の Realtime APIを利用することで、複雑なWebSocketの実装なしで、リアルタイム機能を簡単に追加できる
- パフォーマンス向上
    - クライアント側での定期的なリクエスト送信ではなく、サーバーからのイベント通知を利用するため、リソースを効率的に使用し、パフォーマンスを向上させることができる。

## チャネル
- SupabaseのRealtime APIにおけるチャネル（Channel）は、データのやり取りを行うための基本的な構成単位。チャネルを使うことで、クライアント間でメッセージを送受信したり、データベースの変更をリアルタイムで監視したりすることができる。
### 具体的にできること
- メッセージの送受信  
    - クライアントは、特定のチャネルにメッセージを送信したり、そのチャネル内で送信されたメッセージを受信することができる。
    - メッセージのやり取りは双方向で、リアルタイムに行われる。例えば、チャットアプリのメッセージ交換や、通知システムに使用できる。
- PostgreSQLのデータベース変更を監視
    - Realtime APIでは、PostgreSQLのデータベースに対する変更（INSERT, UPDATE, DELETEなど）をリアルタイムで監視することができる。
    - 例えば、あるテーブルに新しいレコードが挿入された際に、その変更をチャネルを通じて通知することができる。
- 参加者の状態把握
    - チャネルに接続しているクライアントは、リアルタイムで他のクライアントがオンラインかオフラインかを把握することができる。
    - 例えば、チャットアプリで誰がオンライン中かを表示するために使用することができる。
### チャネルの使用方法
- チャネルは、SupabaseのRealtimeクライアントを使って簡単に作成し、メッセージの送受信やデータベース変更の監視を行うことができる。次のコード例では、todosというテーブルで発生したINSERTイベントを監視して、その結果を処理している。

~~~
// Supabaseクライアントの初期化
const { createClient } = require('@supabase/supabase-js')
const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY)

// handleInserts関数（新しいデータが挿入されたときに呼ばれる関数）を定義
const handleInserts = (payload) => {
  // 新しく挿入された行（データ）の情報を表示
  console.log('New row inserted', payload.new)
}

// todosテーブルのINSERTイベントを監視するチャネルを作成
supabase
  .channel('todos')  // チャネル名を指定
  // public.todosテーブルへのINSERTイベントが発生するたびにhandleInserts関数が実行される
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'todos' }, handleInserts)
  .subscribe()  
~~~
- todosテーブルに新しい行が挿入されるたびに、handleInserts関数が呼び出され、その中でpayload.newを使って挿入されたデータを表示する処理。
- payload.new（挿入された新しいデータ）の例
~~~
{
  id: 1,                          
  create_at: '2024-11-20T12:34:56', 
  title: 'Task1',            
  user_id: 001                   
}
~~~

# Realtime APIの使い方
## APIの基本的なフロー
- 接続
  - クライアントがWebSocketを使ってサーバーに接続
- イベント監視
  - クライアントは、特定のテーブルやイベント（例えば、INSERT、UPDATE、DELETE）に対する変更を監視。
- メッセージ送信
  - クライアントはサーバーにメッセージを送信し、サーバーはそのメッセージを他のクライアントに配信。
- データ更新
  - サーバー側でデータが変更されると、その変更がWebSocketを通じてクライアントに通知される。
## 使い方
- サーバーへの接続
  - Realtime APIはWebSocketを使用してサーバーとクライアント間で双方向通信を行う。この接続により、サーバーはクライアントにデータをリアルタイムで送信でき、クライアントはイベントに応じて即座にサーバーにデータを送信することができる。
  - 下記のコードは、ローカルホストのWebSocketサーバーに接続し、クライアントがサーバーからデータをリアルタイムで受信する準備をする。
~~~
import { RealtimeClient } from '@supabase/realtime-js'

const client = new RealtimeClient('ws://localhost:4000/socket', {
  params: { apikey: 'your-api-key' }
})

client.connect()
~~~

- イベントの監視
  - 特定のデータベースのテーブルに対する変更を監視し、それに反応することができる。
  - 例えば、テーブルに新しいデータが追加されたり、更新されたりした場合、それに対応するイベントを受け取ることができる。
  - このコードは、messagesテーブルに新しいメッセージが挿入されたときにそのデータをクライアントに通知する。
~~~
const channel = client.channel('realtime:messages')

channel.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, (payload) => {
  console.log('New message:', payload)
})

channel.subscribe()
~~~

- メッセージの送信
  - サーバーにsend_messageイベントを送信し、サーバーがそのメッセージを他のクライアントにブロードキャストする。
~~~
channel.push('send_message', { text: 'Hello, World!' })
~~~

# 安全性について
- Row Level Security (RLS) ポリシー
  - RLSは、PostgreSQLで特定のテーブルに対して**行単位**でアクセス制御を行うための仕組み。この仕組みによって、ユーザーがアクセスできるデータを制限することができる。
  - 具体的には、テーブルに対するクエリが実行される際、どの行にアクセスできるかを制御する。
  - RLSポリシーを使うことで、どのクライアントがどの操作を行えるかを細かく制御できる。例えば、あるクライアントはメッセージを発信できても、別のクライアントは受信のみ許可することなどが可能になる。
- 制御対象
  - クライアントがチャネルにメッセージを送信できるか。
  - クライアントがチャネルからメッセージを受信できるか。
  - クライアントがチャネルにプレゼンス情報を公開できるか。
  - クライアントが他のクライアントのプレゼンス情報を受信できるか。