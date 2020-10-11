---
layout: post
title:  "OpenAPI3.0(Swagger)レシピリスト"
date:   2019-09-08 01:00:00 +0900
categories: openapi3
---

OpenAPI3.0で「あれ、これどうやるんだっけ？」的なやつをまとめてく記事。

## Bearer トークンの使用を定義する

アクセストークンなどによるBearerスキームをOpenAPI3.0で定義する方法を記載する。

まず、components.securitySchemesにBearerスキームを追加する。
```yaml
components:
  securitySchemes:
    Bearer:
      type: http
      scheme: bearer
      description: Credentials or access token for API
```

APIにBearスキームを利用するための記述をする。
```yaml
paths:
  /users/:id:
    get:
      security:
      - Bearer: []
```

ちなみに、全APIでBearスキームを利用したい場合はルートレベルに記述すればOK。
```yaml
security:
- Bearer: []
```

[サンプル](https://editor.swagger.io/)載せとく。
```yaml
openapi: 3.0.0
info:
  title: SAMPLE API
  description: サンプルなAPI.
  version: 1.0.0
servers:
  - url: https://dev.sample-server.com/v1
    description: Development server
  - url: https://stg.sample-server.com/v1
    description: Staging server
paths:
  /users:
    get:
      tags:
        - users
      summary: ユーザー情報取得API
      description: 全ユーザー情報を返却する。
      responses:
        '200':
          description: OK.
          content:
            application/json:
              schema:
                type:
                  array
                items:
                  $ref: '#/components/schemas/user'
  /users/{:id}:
    get:
      tags:
        - users
      summary: ユーザー情報取得API
      description: ユーザー情報を返却する。
      responses:
        '200':
          description: OK.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/user'
components:
  schemas:
    user:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          example: 1
        name:
          type: string
          example: residenti
  securitySchemes:
    Bearer:
      type: http
      scheme: bearer
      description: 取得したアクセストークンを指定する。
security:
  - Bearer: []
tags:
  - name: users
    description: ユーザー情報取得API
```

### 参考サイト
・[OpenAPI (Swagger) 3.0 で Bearer トークンの使用を定義する](https://riotz.works/articles/lopburny/2019/08/17/describe-bearer-scheme-in-openapi-3/)
