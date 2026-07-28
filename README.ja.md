<!-- i18n: language-switcher -->
[English](README.md) | [日本語](README.ja.md)

# Elasticsearchコネクタ

Elasticsearch用のネイティブIrodoriテーブルコネクタ拡張機能。

このクレートは、コネクタのメタデータ、ネイティブABIのエクスポート、およびIrodori拡張マーケットプレイスで使用されるドライバの実装をパッケージ化しています。

## コネクタ

- 拡張ID: `irodori.elasticsearch`
- エンジンID: `elasticsearch`
- ワイヤープロトコル: `search`
- デフォルトポート: `9200`
- ネイティブABI: `irodori.connector.native.v1`
- ドライバリンク済み: `yes`
- マーケットプレイスの公開範囲: `public`
- パッケージバージョン: `0.1.3`

このパッケージはコネクタのメタデータとネイティブドライバを直接使用しており、デスクトップアダプターのソーススナップショットは必要ありません。

コネクタのメタデータは`connector.config.json`と`irodori.extension.json`に格納されています。
Rustクレートは`src/lib.rs`からネイティブABIをエクスポートし、`irodori-connector-abi`を共有JSON/バッファヘルパーとして使用し、コネクタの動作は`src/driver.rs`に保持しています。

## 接続メタデータ

- エンドポイントモード: `hostPort`, `connectionString`
- トランスポートモード: `direct`, `sshTunnel`, `socks5Proxy`, `httpConnectProxy`, `proxyChain`
- TLS対応: `yes`
- TLS必須（デフォルト）: `no`
- カスタムドライバオプション: `yes`

### エンドポイントフィールド

| フィールド | ラベル | 型 | 必須 |
| --- | --- | --- | --- |
| `host` | ホスト | `string` | yes |
| `port` | ポート | `number` | no |
| `database` | データベース | `string` | no |

## 認証

コネクタはこれらの認証モードを公開しており、クライアントは適切な資格情報フィールドをレンダリングできます。必要に応じて、`options`を通じてドライバ固有またはプロバイダー固有の値も渡せます。

| 認証方法 | ラベル | 種類 | シークレットの用途 |
| --- | --- | --- | --- |
| `none` | 認証なし | `none` | なし |
| `connectionString` | 接続文字列 / DSN | `connectionString` | なし |
| `basic` | ベーシック認証 | `userPassword` | `password` |
| `apiKey` | APIキー | `apiKey` | `token` |
| `bearerToken` | ベアラートークン | `token` | `token` |
| `jwt` | JWTベアラートークン | `token` | `token` |
| `accessToken` | アクセストークン | `token` | `token` |
| `oauth2` | OAuth 2.0 | `oauth2` | `token` |
| `clientCertificate` | クライアント証明書 / mTLS | `certificate` | `privateKey`, `privateKeyPassphrase` |
| `customDriverOptions` | カスタムドライバオプション | `custom` | `password`, `token`, `privateKey`, `privateKeyPassphrase` |

## エクスペリエンスメタデータ

- ドメイン: `search`, `vector`
- 結果ビュー: `searchHits`, `facets`, `json`, `table`, `vectorNeighbors`
- オブジェクトタイプ: `indexes`, `mappings`, `aliases`, `templates`, `analyzers`, `collections`, `vectors`, `payloadFields`, `partitions`, `namespaces`
- インスピレーション元: Kibana Discover、Elasticsearchの集約、ハイブリッド検索、kNN検索

| ワークフロー | 結果ビュー | テンプレート |
| --- | --- | --- |
| ドキュメントの探索 | `searchHits` | `search-query-string` |
| ファセットの分解 | `facets` | `search-facets` |
| ハイブリッド検索 | `searchHits` | `search-hybrid` |
| 類似性検索 | `vectorNeighbors` | `vector-similarity` |
| フィルタ付きANN検索 | `vectorNeighbors` | `vector-filtered` |
| コレクションまたはインデックスの状態 | `table` | `vector-health` |

| テンプレート | ラベル | 言語 | 結果ビュー |
| --- | --- | --- | --- |
| `search-query-string` | クエリ文字列検索 | `json` | `searchHits` |
| `search-facets` | タームファセット | `json` | `facets` |
| `search-hybrid` | ハイブリッドテキストとベクトル検索 | `json` | `searchHits` |
| `vector-similarity` | kNNベクター検索 | `json` | `vectorNeighbors` |
| `vector-filtered` | フィルタ付きkNN検索 | `json` | `vectorNeighbors` |
| `vector-health` | インデックスマッピング | `text` | `json` |

## ネイティブABIコール

| メソッド | 応答 |
| --- | --- |
| `health` | コネクタのヘルス状態、エンジンID、ABIバージョン、ドライバの状態を返します。 |
| `describe` | 埋め込みマニフェストとコネクタ設定を返します。 |
| `manifest` | 生の`irodori.extension.json`を返します。 |
| `config` | 生の`connector.config.json`を返します。 |
| `connect` | ネイティブコネクタ接続を開き、検証します。 |
| `query` | コネクタクエリを実行し、構造化された行またはJSON結果を返します。 |
| `metadata` | スキーマ、テーブル、カラム、インデックス、コレクション、または同等のメタデータを読み取ります。 |
| `close` | キャッシュされたネイティブ接続を閉じて削除します。 |

## 開発

このチェックアウト内のすべての拡張クレートは`../target`を共有しており、依存関係は兄弟リポジトリ間で一度だけコンパイルされます。

```sh
make check
make build
```

リリースパッケージはプラットフォーム固有のネイティブアーティファクトを`dist/native`に配置します。

## ライセンス

0BSD。このプロジェクトはほぼすべての目的で使用、コピー、修正、配布できます。