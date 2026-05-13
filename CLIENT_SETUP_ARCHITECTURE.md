# AutoStore Client Setup Architecture

> Last updated: 2026-04-10 (v2 — download-based, no git required)

## Principle: Server-Driven, Download-Based Setup

All setup logic lives on the **backend server**. The client is a thin wrapper that:

1. Fetches the setup manifest from the server
2. Checks local environment (only Node.js + MySQL needed)
3. Downloads pre-built project bundles (zip) from the server
4. Creates databases and initializes schemas
5. Ready to run — no git, no npm install, no compilation

**Why not git clone?** Our repos are private. Requiring git + SSH keys on every client machine is terrible UX. Instead, the backend server builds projects from git and serves pre-built zip bundles via HTTP.

---

## Architecture

```
┌────────────────────┐              ┌──────────────────────────────────┐
│  AutoStore Mac     │              │     autostore-backend            │
│  (or any client)   │              │     (remote server)              │
│                    │              │                                  │
│  Only needs:       │   HTTP/JWT   │  Has git access to all repos     │
│  - Node.js         │◀───────────▶│  Builds + zips projects          │
│  - MySQL           │              │  Serves manifests + bundles      │
│                    │              │  Hosts DB schemas                │
│  NO git needed     │              │                                  │
│  NO SSH keys       │              │                                  │
│  NO npm install    │              │                                  │
│  NO TypeScript     │              │                                  │
└────────────────────┘              └──────────────────────────────────┘
```

---

## API Endpoints

### `GET /api/setup/manifest`
Returns the full setup manifest — what the client needs to check and install.

```json
{
  "version": "1.0.0",
  "updated_at": "2026-04-10T00:00:00Z",

  "client_requirements": [
    {
      "name": "node",
      "display_name": "Node.js",
      "check_command": "node --version",
      "min_version": "20.0.0",
      "install_instructions": {
        "mac": "Visit https://nodejs.org or run: brew install node@20",
        "manual_url": "https://nodejs.org/en/download/"
      },
      "required": true
    },
    {
      "name": "mysql",
      "display_name": "MySQL",
      "check_command": "mysql --version",
      "min_version": "8.0.0",
      "install_instructions": {
        "mac": "brew install mysql && brew services start mysql",
        "manual_url": "https://dev.mysql.com/downloads/mysql/"
      },
      "required": true
    }
  ],

  "optional_dependencies": [
    {
      "name": "chromium",
      "display_name": "Chromium Browser",
      "check_command": "ls /Applications/Chromium.app || ls '/Applications/Google Chrome.app'",
      "install_instructions": {
        "mac": "npx playwright install chromium"
      },
      "note": "Required for browser automation tasks (1688 scraping, platform uploads)"
    }
  ],

  "projects": [
    {
      "name": "1688_scrapper",
      "display_name": "1688 Scraper",
      "description": "Product discovery and scraping pipeline",
      "download_endpoint": "/api/setup/download/1688_scrapper",
      "database": "1688_source",
      "size_mb": null
    },
    {
      "name": "aliexpress",
      "display_name": "AliExpress",
      "description": "Excel generation and bulk upload to AliExpress Seller Center",
      "download_endpoint": "/api/setup/download/aliexpress",
      "database": "aliexpress_autostore",
      "size_mb": null
    },
    {
      "name": "ebay",
      "display_name": "eBay",
      "description": "CSV generation and Seller Hub upload",
      "download_endpoint": "/api/setup/download/ebay",
      "database": "ebay_autostore",
      "size_mb": null
    },
    {
      "name": "etsy",
      "display_name": "Etsy",
      "description": "SEO optimization and API listing",
      "download_endpoint": "/api/setup/download/etsy",
      "database": "etsy_autostore",
      "size_mb": null
    },
    {
      "name": "amazon",
      "display_name": "Amazon",
      "description": "Flat file generation and Seller Central upload",
      "download_endpoint": "/api/setup/download/amazon",
      "database": null,
      "size_mb": null
    }
  ],

  "databases": [
    {
      "name": "1688_source",
      "charset": "utf8mb4",
      "collation": "utf8mb4_unicode_ci",
      "schema_endpoint": "/api/setup/schema/1688_source"
    },
    {
      "name": "aliexpress_autostore",
      "charset": "utf8mb4",
      "collation": "utf8mb4_unicode_ci",
      "schema_endpoint": "/api/setup/schema/aliexpress_autostore"
    },
    {
      "name": "ebay_autostore",
      "charset": "utf8mb4",
      "collation": "utf8mb4_unicode_ci",
      "schema_endpoint": "/api/setup/schema/ebay_autostore"
    },
    {
      "name": "etsy_autostore",
      "charset": "utf8mb4",
      "collation": "utf8mb4_unicode_ci",
      "schema_endpoint": "/api/setup/schema/etsy_autostore"
    }
  ]
}
```

### `GET /api/setup/schema/:dbName`
Returns CREATE TABLE SQL for a specific database.

### `GET /api/setup/download/:project`
Returns a pre-built zip bundle for a project. Includes `dist/`, `node_modules/`, config files. Client just unzips and runs.

### `GET /api/setup/client-version`
Returns latest Mac client version + download URL for auto-update checks.

---

## Client Setup Flow (Mac App Setup Wizard)

### Screen 1: Check Environment
```
环境检查

✓ Node.js v22.0.0 (需要 20+)
✗ MySQL — 未安装
  → brew install mysql && brew services start mysql

[重新检查]  [继续]
```

### Screen 2: Configure MySQL
```
数据库配置

MySQL 主机: localhost
端口: 3306
用户名: root
密码: ********

[测试连接]  [继续]
```

### Screen 3: Download Projects
```
下载项目

↓ 1688 Scraper          ████████████ 100%  ✓
↓ AliExpress            ████████░░░░  67%
↓ eBay                  等待中...
↓ Etsy                  等待中...
↓ Amazon                等待中...

安装目录: ~/projects/autostore/
```

### Screen 4: Initialize Databases
```
初始化数据库

→ 创建 1688_source            ✓
→ 创建 aliexpress_autostore   ✓
→ 创建 ebay_autostore         ✓
→ 创建 etsy_autostore         ✓
→ 创建表结构                   ✓

[完成]
```

### Screen 5: Done
```
设置完成！

✓ 5 个项目已下载
✓ 4 个数据库已初始化
✓ 所有依赖已就绪

项目目录: ~/projects/autostore/

[开始使用]
```

---

## Server-Side Build Pipeline

The backend server (which has git access) runs a build job:

1. `git pull` each repo
2. `npm install` + `tsc` build
3. `zip -r` the project (dist/ + node_modules/ + config)
4. Store in `/autostore/bundles/` for serving via download endpoint
5. Rebuild on demand or on schedule

---

## Rules

1. **Client only needs Node.js + MySQL** — no git, no SSH keys, no npm, no TypeScript
2. **Server builds and bundles** — pre-built zips with dist/ + node_modules/
3. **Schemas are server-hosted** — CREATE TABLE SQL served via API
4. **Download over HTTP** — JWT-authenticated, no special tooling
5. **Idempotent** — re-running setup is safe
6. **Chinese UI** — all setup wizard text in Simplified Chinese
7. **Auto-update** — client checks server for new versions on launch
