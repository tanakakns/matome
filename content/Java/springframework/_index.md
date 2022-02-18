---
title: "Spring Framework"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 2
---

Spring Framework もといほぼ SpringBoot ベースのメモ。

1. [プロジェクト作成](#1-プロジェクト作成)
2. [アノテーション](#2-アノテーション)
3. [Beanスコープ](#3-Beanスコープ)
4. [設定](#4-設定)
5. [プロジェクト構成](#5-プロジェクト構成)

## 1. プロジェクト作成

[spring initialzr](https://start.spring.io/) で SpringBoot プロジェクトを作成するのがよい。  
以下のような依存を追加する。

- ToDo

## 2. アノテーション

すぐ忘れるのでメモ。分類はざっくり。

## 2.1. Core系

|アノテーション          |付与箇所|説明|
|:----------------|:------------|:----|
|@Configuration|クラス|Java Based Configurationクラスとなる。Bean 定義クラスとして@Bean メソッドなどを宣言。|
|@ComponentScan|クラス|@Configuration 付与されたJava Based Configurationクラスに付与し、コンポーネントスキャンを有効化。|
|@Bean         |||
|@Value        |||
|@Component    |クラス|DIコンテナにbeanとして登録するクラスへ付与。|
|@Service      |クラス|@Component と同じだが、明確にService層示す際に利用。|
|@Repository   |クラス|@Component と同じだが、明確にPersistence層（DAO等のDBアクセスを行うクラス）示す際に利用。|
|@Controller   |クラス|@Component と同じだが、明確にSpringMVCでControllerの役割となるクラスに利用。|
|@Scope        |クラス|beanのスコープを設定。|
|@Transactional|クラス、メソッド|トランザクション境界を設定。|
|@Autowired    |メソッド、フィールド|フィールドの型（インターフェース）の実装クラスのbeanを自動的にインジェクション。|
|@Qualifier    |メソッド、フィールド|@Autowiredと併用し、bean名を指定したい場合に使用。|
|@Value        |フィールド|application.yamlなどで指定した値を代入。|
|@Required     |メソッド|Setterメソッドに付与し、インジェクションが必須であることを示す。|

## 2.2. SpringMVC系

|アノテーション     |付与箇所|説明|
|:----------------|:------------|:----|
|@Controller      |クラス        |MVCコントローラ。|
|@RestController  |クラス        |REST用コントローラ。@Controller + @ResponseBodyと同じ。|
|@RequestMapping  |クラス、メソッド|URI（value）、HTTPメソッド（method）、HTTPヘッダ（headers）、リクエストパラメータ（params）などでコントローラへのルーティングを設定。|
|@RequestParam    |メソッド       |URLのクエリパラメータをメソッド引数に注入。|
|@PathVariable    |メソッド       |URLのパスパラメータをメソッド引数に注入。|
|@RequestHeader   |メソッド       |リクエストヘッダをメソッド引数に注入。|
|@RequestBody     |メソッド       |リクエストボディをメソッド引数に注入。|
|@ResponseBody    |メソッド       |return するオブジェクトをHTTPレスポンスボディに変換。|
|@ResponseStatus  |メソッド       |HTTPレスポンスステータスを指定。|
|@ExceptionHandler|メソッド       |クラス内で発生した例外を処理するメソッドに付与。|
|@CookieValue     |メソッド       |特定の Cookie を処理するメソッドに付与。|
|@InitBinder      |||
|@ModelAttribute  |||

## 2.3. SpringBoot系

|アノテーション          |付与箇所|説明|
|:----------------|:------------|:----|
|@SpringBootApplication|クラス|@Configuration 、 @EnableAutoConfiguration 、 @ComponentScan をまとめたもの。SpringBootアプリケーション起動クラスに付与する。|
|@EnableAutoConfiguration|クラス|SpringBoot の自動構成メカニズムを有効化。|

## 2.4. Test系

|アノテーション          |付与箇所|説明|
|:----------------|:------------|:----|
|@RunWith(SpringRunner.class) |クラス|Spring BootでJUnitテストするときはコレをつける。|
|@SpringBootTest|クラス|SpringのJava/XML Based Configurationなどの設定を読み込んでくれる。|
|@AutoConfigureMockMvc|?|コントローラ層のモック（ MockMvc ）を作成でき、これでコントローラのJUnitテストが可能になる。|
|@WebMvcTest(HelloController.class)|?|上記に加えてコントローラクラスを指定することもできる。|
|@MockBean|?|コントローラ内でDIされるモジュールのモック（Mockito）を作成できる。|

## 3. Beanスコープ

|スコープ|説明|
|:----------------|:------------|
|singleton|(Default) ApplicationContext に 1 つインスタンス化。|
|prototype|Bean 取得毎にインスタンス化。スレッドアンセーフの場合に利用。|
|request|HTTP リクエスト毎にインスタンス化。|
|session|HTTPセッション単位でインスタンス化。|
|application|サーブレットのコンテキスト単位でインスタンス化。warの単位なイメージ。|
|websocket|WebSocketのライフサイクル単位でインスタンス化。|
|custom|上記以外の独自カスタムスコープ。|

## 4. 設定

Spring の設定には

- Java Based Configuration
- XML Based Configuration

の 2 種類あるが、もはや前者しか使わないと思われるため前者に限定して記載する。

- Spring設定クラス（ `src/main/java/com/pepese/sample/config/AppConfig.java` などに配置）
  - DIやAOPなどの設定
  - `@Configuration` アノテーションを付与
- SpringMVC設定クラス（ `src/main/java/com/pepese/sample/config/WebMvcConfig.java` などに配置）
  - コンテンツネゴシエーションなどSpring MVCのDispatcherServletに関する設定
  - `WebMvcConfigurer` を継承
    - Spring4 -> 5、SpringBoot1 -> 2 にかけて `WebMvcConfigurerAdapter` が非推奨となった
  - `@Configuration` アノテーションを付与すること
    - SpringBootの場合、 `@EnableWebMvc` を付与する必要はない
    - SpringBootを利用しないSpring MVCアプリケーションを作成する場合には付与する
- SpringSecurity設定クラス（ `src/main/java/com/pepese/sample/config/WebSecurityConfig.java` などに配置）
  - `@Configuration` 、 `@EnableWebSecurity` アノテーションを付与
  - `WebSecurityConfigurerAdapter` を継承
  - `configure` メソッドなどをオーバーライドして利用
- SpringSession設定クラス（ `src/main/java/com/pepese/sample/config/SessionConfig.java` などに配置）
  - `@Configuration` 、 `@EnableRedisHttpSession` アノテーションを付与
  - ConnectionFactory などを生成するメソッドを実装して利用

## 5. プロジェクト構成

CleanArchitecture 風なプロジェクト構成。

- `src/main/java/com/pepese/sample/Application.java` ： SpringBoot 起動クラス
- `src/main/java/com/pepese/sample/config/` ： Java Based Configuration クラス
- `src/main/java/com/pepese/sample/adapter/` ： コントローラとか
- `src/main/java/com/pepese/sample/domain/` ： ドメインとなる Bean クラス
- `src/main/java/com/pepese/sample/infrastructure/` ： リポジトリとか
- `src/main/java/com/pepese/sample/usecase/` ： ユースケースとなる Service クラス
- 残りメモ
  - ロガーは？