---
title: 使用 mysql2 连接 TiDB
summary: 学习如何使用 Ruby 的 mysql2 连接 TiDB。本教程提供了使用 mysql2 gem 与 TiDB 协同工作的 Ruby 示例代码片段。
---

# 使用 mysql2 连接 TiDB

TiDB 是一个与 MySQL 兼容的数据库，而 [mysql2](https://github.com/brianmario/mysql2) 是 Ruby 中最受欢迎的 MySQL 驱动之一。

在本教程中，你将学习如何使用 TiDB 和 mysql2 完成以下任务：

- 设置你的环境。
- 使用 mysql2 连接你的 TiDB 集群。
- 构建并运行你的应用程序。你还可以选择查看基本 CRUD 操作的[示例代码片段](#示例代码片段)。

> **注意：**
>
> 本教程适用于 TiDB Serverless、TiDB Dedicated 以及本地部署的 TiDB。

## 先决条件

要完成本教程，你需要：

- 在你的机器上安装 [Ruby](https://www.ruby-lang.org/en/) >= 3.0
- 在你的机器上安装 [Bundler](https://bundler.io/)
- 在你的机器上安装 [Git](https://git-scm.com/downloads)
- 运行一个 TiDB 集群

如果你还没有 TiDB 集群，可以按照以下方式创建：

- （推荐方式）参考[创建 TiDB Serverless 集群](/develop/dev-guide-build-cluster-in-cloud.md#第-1-步创建-tidb-serverless-集群)，创建你自己的 TiDB Cloud 集群。
- 参考[部署本地测试 TiDB 集群](/quick-start-with-tidb.md#部署本地测试集群)或[部署正式 TiDB 集群](/production-deployment-using-tiup.md)，创建本地集群。

## 运行示例应用程序并连接到 TiDB

本节将演示如何运行示例应用程序代码并连接到 TiDB。

### 步骤 1：克隆示例应用程序仓库

在你的终端窗口中运行以下命令来克隆示例代码仓库：

```shell
git clone https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart.git
cd tidb-ruby-mysql2-quickstart
```

### 步骤 2：安装依赖

运行以下命令安装示例应用程序所需的包（包括 `mysql2` 和 `dotenv`）：

```shell
bundle install
```

<details>
<summary><b>为现有项目安装依赖</b></summary>

对于你的现有项目，运行以下命令安装这些包：

```shell
bundle add mysql2 dotenv
```

</details>

### 步骤 3：配置连接信息

根据你选择的 TiDB 部署选项，连接到你的 TiDB 集群。

<SimpleTab>
<div label="TiDB Serverless">

1. 在 TiDB Cloud 的 [**Clusters**](https://tidbcloud.com/console/clusters) 页面中，点击你的目标集群的名称，进入集群的 **Overview** 页面。

2. 点击右上角的 **Connect**。会显示一个连接对话框。

3. 确保连接对话框中的配置与你的操作环境匹配。

   - **Endpoint Type** 设置为 `Public`。
   - **Connect With** 设置为 `General`。
   - **Operating System** 与你运行应用程序的操作系统相匹配。

4. 如果你还没有设置密码，点击 **Create password** 生成一个随机密码。

5. 运行以下命令复制 `.env.example` 并将其重命名为 `.env`：

    ```shell
    cp .env.example .env
    ```

6. 编辑 `.env` 文件，按照以下方式设置环境变量，并将连接对话框中的连接参数替换为相应的占位符 `<>`：

    ```dotenv
   DATABASE_HOST=<host>
   DATABASE_PORT=4000
   DATABASE_USER=<user>
   DATABASE_PASSWORD=<password>
   DATABASE_NAME=test
   DATABASE_ENABLE_SSL=true
    ```

   > **注意：**
   >
   > 对于 TiDB Serverless，当使用 Public Endpoint 时，必须通过 `DATABASE_ENABLE_SSL` 启用 TLS 连接。

7. 保存 `.env` 文件。

</div>
<div label="TiDB Dedicated">

1. 导航到 [**Clusters**](https://tidbcloud.com/console/clusters) 页面，然后点击你的目标集群的名称，进入其概览页面。

2. 点击右上角的 **Connect**。会显示一个连接对话框。

3. 点击 **Allow Access from Anywhere**，然后点击 **Download TiDB cluster CA** 以下载 CA 证书。

   关于如何获取连接字符串的更多细节，请参考 [TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)。

4. 运行以下命令复制 `.env.example` 并将其重命名为 `.env`：

    ```shell
    cp .env.example .env
    ```

5. 编辑 `.env` 文件，按照以下方式设置环境变量，并将连接对话框中的连接参数替换为相应的占位符 `<>`：

    ```dotenv
    DATABASE_HOST=<host>
    DATABASE_PORT=4000
    DATABASE_USER=<user>
    DATABASE_PASSWORD=<password>
    DATABASE_NAME=test
    DATABASE_ENABLE_SSL=true
    DATABASE_SSL_CA=<downloaded_ssl_ca_path>
    ```

   > **注意：**
   >
   > 当使用 Public Endpoint 连接到 TiDB Dedicated 集群时，建议启用 TLS 连接。
   >
   > 要启用 TLS 连接，将 `DATABASE_ENABLE_SSL` 修改为 `true`，并使用 `DATABASE_SSL_CA` 指定从连接对话框下载的 CA 证书的文件路径。

6. 保存 `.env` 文件。

</div>
<div label="本地部署的 TiDB">

1. 运行以下命令复制 `.env.example` 并将其重命名为 `.env`：

    ```shell
    cp .env.example .env
    ```

2. 编辑 `.env` 文件，按照以下方式设置环境变量，并将连接对话框中的连接参数替换为相应的占位符 `<>`：

    ```dotenv
    DATABASE_HOST=<host>
    DATABASE_PORT=4000
    DATABASE_USER=<user>
    DATABASE_PASSWORD=<password>
    DATABASE_NAME=test
    ```

   如果你在本地运行 TiDB，那么默认的主机地址是 `127.0.0.1`，密码为空。

3. 保存 `.env` 文件。

</div>
</SimpleTab>

### 步骤 4：运行代码并检查结果

运行以下命令来执行示例代码：

```shell
ruby app.rb
```

如果连接成功，控制台将输出 TiDB 集群的版本，如下所示：

```
🔌 Connected to TiDB cluster! (TiDB version: 5.7.25-TiDB-v7.1.0)
⏳ Loading sample game data...
✅ Loaded sample game data.

🆕 Created a new player with ID 12.
ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
🚮 Deleted 1 player data.
```

## 示例代码片段

你可以参考以下示例代码片段来完成你自己的应用程序开发。

要获取完整的示例代码以及如何运行它，请查看 [tidb-samples/tidb-ruby-mysql2-quickstart](https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart) 仓库。

### 使用连接选项连接到 TiDB

以下代码使用环境变量中定义的选项建立到 TiDB 的连接：

```ruby
require 'dotenv/load'
require 'mysql2'
Dotenv.load # Load the environment variables from the .env file

options = {
  host: ENV['DATABASE_HOST'] || '127.0.0.1',
  port: ENV['DATABASE_PORT'] || 4000,
  username: ENV['DATABASE_USER'] || 'root',
  password: ENV['DATABASE_PASSWORD'] || '',
  database: ENV['DATABASE_NAME'] || 'test'
}
options.merge(ssl_mode: :verify_identity) unless ENV['DATABASE_ENABLE_SSL'] == 'false'
options.merge(sslca: ENV['DATABASE_SSL_CA']) if ENV['DATABASE_SSL_CA']
client = Mysql2::Client.new(options)
```

> **注意：**
>
> 对于 TiDB Serverless，当使用 Public Endpoint 时，必须通过 `DATABASE_ENABLE_SSL` 启用 TLS 连接，但是你**不必**通过 `DATABASE_SSL_CA` 指定 SSL CA 证书，因为 mysql2 gem 会按照特定的顺序搜索现有的 CA 证书，直到找到一个文件。

### 插入数据

以下查询创建一个具有两个字段的玩家，并返回 `last_insert_id`：

```ruby
def create_player(client, coins, goods)
  result = client.query(
    "INSERT INTO players (coins, goods) VALUES (#{coins}, #{goods});"
  )
  client.last_id
end
```

有关更多信息，请参考 [插入数据](/develop/dev-guide-insert-data.md)。

### 查询数据

以下查询通过 ID 返回特定玩家的记录：

```ruby
def get_player_by_id(client, id)
  result = client.query(
    "SELECT id, coins, goods FROM players WHERE id = #{id};"
  )
  result.first
end
```

有关更多信息，请参考 [查询数据](/develop/dev-guide-get-data-from-single-table.md)。

### 更新数据

以下查询通过 ID 更新特定玩家的记录：

```ruby
def update_player(client, player_id, inc_coins, inc_goods)
  result = client.query(
    "UPDATE players SET coins = coins + #{inc_coins}, goods = goods + #{inc_goods} WHERE id = #{player_id};"
  )
  client.affected_rows
end
```

有关更多信息，请参考 [更新数据](/develop/dev-guide-update-data.md)。

### 删除数据

以下查询删除特定玩家的记录：

```ruby
def delete_player_by_id(client, id)
  result = client.query(
    "DELETE FROM players WHERE id = #{id};"
  )
  client.affected_rows
end
```

有关更多信息，请参考 [删除数据](/develop/dev-guide-delete-data.md)。

## 最佳实践

默认情况下，mysql2 gem 可以按照特定的顺序搜索现有的 CA 证书，直到找到一个文件。

1. 对于 Debian、Ubuntu、Gentoo、Arch 或 Slackware，路径为 `/etc/ssl/certs/ca-certificates.crt`
2. 对于 RedHat、Fedora、CentOS、Mageia、Vercel 或 Netlify，路径为 `/etc/pki/tls/certs/ca-bundle.crt`
3. 对于 OpenSUSE，路径为 `/etc/ssl/ca-bundle.pem`
4. 对于 macOS 或 Alpine（docker 容器），路径为 `/etc/ssl/cert.pem`

虽然可以手动指定 CA 证书路径，但在多环境部署场景中，这可能会造成很大的不便，因为不同的机器和环境可能会在不同的位置存储 CA 证书。因此，建议将 `sslca` 设置为 `nil`，以便在不同环境中灵活且方便地部署。

## 下一步

- 从 [mysql2 的文档](https://github.com/brianmario/mysql2#readme)中了解更多关于 mysql2 驱动的使用情况。
- 通过 [开发者指南](/develop/dev-guide-overview.md) 中的章节，了解 TiDB 应用开发的最佳实践，例如：[插入数据](/develop/dev-guide-insert-data.md)、[更新数据](/develop/dev-guide-update-data.md)、[删除数据](/develop/dev-guide-delete-data.md)、[查询数据](/develop/dev-guide-get-data-from-single-table.md)、[事务](/develop/dev-guide-transaction-overview.md) 和 [SQL 性能优化](/develop/dev-guide-optimize-sql-overview.md)。
- 通过专业的 [TiDB 开发者课程](https://www.pingcap.com/education/) 学习，并在通过考试后获得 [TiDB 认证](https://www.pingcap.com/education/certification/)。

## 需要帮助？

在 [Discord](https://discord.gg/vYU9h56kAX) 频道上提问。