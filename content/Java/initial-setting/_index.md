---
title: "初期設定"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 1
---

Javaの初期設定周りについて。

1. [Javaのインストール](#1-Javaのインストール)
2. [Mavenのインストール](#2-Mavenのインストール)
3. [プロジェクトの作成](#3-プロジェクトの作成)
4. [VS Code](#4-vs-code)
5. [docker によるマルチステージビルド](#5-docker-によるマルチステージビルド)

<!--more-->

## 1. Javaのインストール

asdf でインストールする。  
ここでは Amazon ディストリビューションである Corretto を利用する。

```bash
$ asdf plugin add java
$ asdf install java corretto-17.0.2.8.1
$ asdf global java corretto-17.0.2.8.1
```

asdf でインストールした場合、 `JAVA_HOME` の設定には専用のスクリプトが用意されている。（参考： [JAVA_HOME](https://github.com/halcyon/asdf-java#java_home) ）  
シェルの種類に合わせて `.zshenv` などに以下を追記する。

```bash
# To set JAVA_HOME in your shell's initialization add the following:
. ~/.asdf/plugins/java/set-java-home.bash

# For zsh shell, instead use:
. ~/.asdf/plugins/java/set-java-home.zsh

# For fish shell, instead use:
. ~/.asdf/plugins/java/set-java-home.fish

# For xonsh shell, instead use:
source ~/.asdf/plugins/java/set-java-home.xsh
```

```bash
$ exec $SHELL -l
$ java --version
openjdk 17.0.2 2022-01-18 LTS
OpenJDK Runtime Environment Corretto-17.0.2.8.1 (build 17.0.2+8-LTS)
```

## 2. Mavenのインストール

同じく asdf を利用する。

```bash
$ asdf plugin add maven
$ asdf install maven 3.8.4
$ asdf global maven 3.8.4
$ mvn -v
Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
Maven home: /Users/tanakakns/.asdf/installs/maven/3.8.4
Java version: 17.0.2, vendor: Amazon.com Inc., runtime: /Users/tanakakns/.asdf/installs/java/corretto-17.0.2.8.1
Default locale: ja_JP, platform encoding: UTF-8
OS name: "mac os x", version: "12.1", arch: "x86_64", family: "mac"
```

## 3. プロジェクトの作成

通常の Java プロジェクトを作成する場合は以下のコマンドで作成する。

```bash
$ mvn archetype:create -DgroupId=com.example -DartifactId=sample
```

SpringBoot のプロジェクトを作成する場合は [spring initialzr](https://start.spring.io/) を使用すると便利。  
各種設定を入力した後に GENERATE すると SpringBoot の雛形プロジェクトをダウンロードできる。  
最低限 Spring Web を Dependencies として追加しておけば REST API アプリケーションを作成できる。  
実行は以下。

```bash
$ mvn spring-boot:run
```

## 4. VS Code

ToDo  
以下の拡張機能を入れる。

- Extension Pack for Java など Microsoft 製のやつ
- Spring Boot Tools など Pivotal 製のやつ
- Spring Boot Dashboard など Microsoft 製のやつ
- Lombok Annotations Support for VS Code
