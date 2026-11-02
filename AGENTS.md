# readest Agent 規則

本檔是 readest 的 agent / Codex 專案慣例檔。修改程式、文件、測試或發版前，先依照這裡的規則檢查。進入子專案時，也要讀取該目錄的 `AGENTS.md`；主程式另有 `apps/readest-app/AGENTS.md`、`LESSONS.md`、`.agents/rules/` 與 `DESIGN.md`。

## 專案定位

- Readest 是跨平台電子書閱讀器，支援 Android、iOS、Windows、macOS、Linux 與 Web。
- 本庫是 pnpm monorepo；主程式位於 `apps/readest-app/`，使用 Next.js／React／TypeScript 與 Tauri v2／Rust，電子書渲染沿用 `packages/foliate-js/`。
- 本機 App 建置範圍預設僅限 `apps/readest-app/` 的 Android 版本；除非使用者明確要求，不建置 Windows、macOS、Linux、iOS 或獨立 Web 發布產物。Android 建置所需的前端與 Rust 編譯仍須完成。
- 自用 Android App 識別碼固定為 `tw.idv.samliao.readest`，由 `apps/readest-app/src-tauri/tauri.android.conf.json` 覆寫；Android Gradle 的 applicationId、namespace 與主程式 package 必須一致。
- Android 原生整合位於 `apps/readest-app/src-tauri/`，不是獨立的 Compose 或 2FA 驗證器專案；另有 KOReader、Calibre 等整合子專案。

## 文件語言

- 新增的自用說明、交接與開發筆記以正體中文為主；既有上游公開文件沿用原本語言與格式，不為單一修改整份翻譯。
- 使用者介面文字沿用英文作為翻譯鍵的 i18n 模式，依 `apps/readest-app/docs/i18n.md` 更新相關語系。
- 修改功能、權限、建置需求或發版流程時，同步檢查 `README.md`、`apps/readest-app/release-notes.json` 與相關文件。Android 商店 metadata 沿用 `fastlane/` 的既有配置，不新增不存在的 `metadata/en-US/` 慣例。

## 版本與建置來源

- App 版本以 `apps/readest-app/package.json` 的 `version` 為準；`src-tauri/tauri.conf.json` 的 `version` 指向該檔。
- Android `versionName`／`versionCode` 由 Tauri 建置流程傳入 `tauri.properties`，Gradle 透過 `tauri.android.versionName`／`tauri.android.versionCode` 讀取；不要把 Gradle fallback 或 Rust crate 版本當成 App 版本。
- Node、pnpm、Rust、JDK、Android SDK／NDK 與 Gradle 需求以專案設定、README 及 `.github/workflows/release.yml` 為準；不要在本檔重複寫死易過期的版本。
- Android Gradle 專案位於 `apps/readest-app/src-tauri/gen/android/`，本機可能尚未完整初始化；使用既有 Tauri 流程，不假設根目錄存在 `app/` 或 `gradlew.bat`。
- 調整版本、SDK、工具鏈或依賴時，同步檢查相關建置文件、鎖定檔及 CI。交付 APK、release 資料夾與說明中的版本必須一致。
- 若準備 commit、push 或 release，且上次 GitHub release 後已有使用者可見變更，但 App 版本仍等於最新 release，先提醒「版號尚未調整」，詢問是否先調整；不得自行猜測或提升版本。無法取得最新 release 時明確說明。

## 本機簽章設定

- 本機 release 簽章資料由 `D:\project\apkKey\readest.local.ps1` 管理；keystore 為 `D:\project\apkKey\readest.jks`。兩者不進版本庫，不顯示腳本內容或密碼。
- key alias 與密碼以本機設定為準，不得沿用其他專案金鑰。
- 目前 Android Gradle 實作讀取 `src-tauri/gen/android/keystore.properties` 的 `storeFile`、`keyAlias`、`password`，不是直接讀取 `SIGNING_*` 環境變數。載入本機腳本後，先確認它是否已接通此簽章配置；不得宣稱載入腳本就已完成簽章。
- APK 建置從主程式目錄執行 Tauri；環境與前置步驟依 README 與 release workflow：

```powershell
. "D:\project\apkKey\readest.local.ps1"
Push-Location "D:\github\readest\apps\readest-app"
try {
    pnpm tauri android build --apk
} finally {
    Pop-Location
}
```

- 不要把簽章密碼、`keystore.properties`、keystore、`*.jks`、`*.keystore`、`local.properties` 或 APK/AAB 提交進版本庫。APK 簽章與 Tauri updater 簽章是不同用途，依交付管道分別驗證。

## 實作原則

- 優先沿用現有模式；不要為單一需求新增抽象層、外掛系統或大重構。
- bug fix 要找共用根因，先查呼叫者及平台共用路徑，不只修單一症狀。
- 修改書庫、閱讀進度、註記、匯入／匯出、同步或背景工作時，先理解讀寫與遷移流程，檢查失敗、取消、重試及衝突路徑，避免遺失或覆蓋使用者資料。
- 不將帳號 token、服務金鑰、簽章資料、私人書籍內容或使用者識別資料寫入日誌、文件、測試 fixture 或版本庫；測試使用公開授權或假資料。
- UI 沿用現有 React 元件及設定 primitives，遵守 `DESIGN.md`；檢查深色模式、電子紙、RTL、無障礙與行動裝置安全區域。
- 建立 worktree 時使用專案的 `pnpm worktree:new`，不要直接使用 `git worktree add`，以免漏掉子模組、依賴與必要資產。

## 驗證

- 主程式驗證遵循 `apps/readest-app/.agents/rules/verification.md`，行為變更先加可重現的測試。以下從主程式目錄執行：

```powershell
Set-Location "D:\github\readest\apps\readest-app"
pnpm test --run
pnpm lint
```

- 修改 Rust 時另跑 `pnpm fmt:check`、`pnpm clippy:check`、`pnpm test:rust`；修改 KOReader Lua 或其原生 helper 時，依子專案規則執行對應測試。
- UI／渲染／手勢變更補做相關瀏覽器或實機測試；Android 可使用 `pnpm test:android`（需要 Bash、adb、已安裝的開發版與連線裝置）。腳本略過時不得視為實機驗證通過。
- 影響 Android 原生程式、manifest、資源或建置設定時，以 Tauri 流程建置 Android release 版驗證；不額外建 dev／debug 版，不以單獨的 Gradle 任務取代前端與 Rust 整合建置。
- Android 開發需要 SDK／NDK、JDK 與對應 Rust targets；依 README／CI 設定 `ANDROID_HOME`、`NDK_HOME` 等。若使用 `local.properties`，不得提交。
- 純文件變更檢查內容、路徑、命令與差異即可；缺少依賴、裝置或工具時，說明未執行的檢查及原因，不宣稱通過。

## Commit 前檢查

每次 commit 前都要做：

1. 檢查程式行為是否與 README、release notes、相關 metadata 及交接文件相符。
2. 若改到使用者可見功能、權限、建置或資料保存行為，更新相關文件，不為無關修改增加發版內容。
3. 依「版本與建置來源」評估版號；不把每次 commit 都當成一次發版。
4. 若此次準備交付，確認 App 版本、說明與預計 release 版本一致。
5. 執行改動所需驗證；無法執行時，在回覆或 commit message 中說明原因。
6. 檢查 `git diff` 與 staged diff，不 stage 或 commit 使用者未要求處理的無關改動。
7. 完成上述檢查後，才可以 stage 與 commit；是否執行 commit 仍以使用者授權為準。

## Push 前檢查

每次 push 前都要做：

1. 確認完成 Commit 前檢查，並手動執行目前改動需要的測試與建置；不使用 pre-push 自動檢查鉤子，不要套用不存在的根目錄 Gradle 任務。
2. 確認推送的分支、remote 與提交範圍正確，且已取得使用者授權。
3. 一般原始碼 push 不要求先產生 APK；若此次同時準備交付或發版，才建立 `release/[版本]/release-notes.md` 並檢查對應產物。
4. 若已有本次交付 APK，確認它由目前程式建置，固定放在 `release/[版本]/release.apk`，並核對實際版本與簽章。
5. 不 push APK、AAB、keystore、`keystore.properties`、`local.properties`、build cache 或其他本機產物；GitHub release 附件另依 Release 前檢查處理。

## Release 前檢查

每次 release 前都要做：

1. 確認 `release/[版本]/release-notes.md` 存在，內容為本次版本，並與 `apps/readest-app/release-notes.json` 的相關內容一致。
2. Android APK 交付時，確認 `release/[版本]/release.apk` 存在，由預計發版的提交及配置重新建置；有未提交變更時先說明產物對應範圍。
3. 確認 APK 的 `versionName` 與 App 版本一致，`versionCode` 符合目標管道的遞增要求。
4. 驗證 APK 的最低 SDK、target SDK、applicationId、ABI、正式簽章與必要語系；確認 FOSS／Google Play 等建置 flavor 正確。
5. 說明已跑與未跑的測試；不得用舊 APK 代表目前程式。
6. 經使用者授權發布 GitHub release 時，使用 `release/[版本]/release-notes.md` 作為 description，並附上 `release/[版本]/release.apk`。
7. 其他平台或商店發布沿用既有 workflow／腳本與產物規格，不將 APK 規則套用到所有平台。

## Release 輸出慣例

- 使用者要求「建置 App／APK」時，一律直接建置正式簽章的 release APK；不先建 dev／debug 版，不使用 `pnpm dev-android`、`--debug` 或 `--features devtools`。除非使用者明確要求開發版，否則不得以開發版替代交付。
- 本機建置完成的 Android APK 必須放在 repo 最外層的 `D:\github\readest\release\[版本]\release.apk`（相對 repo 根目錄為 `release/[版本]/release.apk`），不可放在 `apps/readest-app/release/`。即使從 `apps/readest-app/` 執行建置，也必須將本次產物複製到此固定位置。
- 本機 release 說明固定路徑：`release/[版本]/release-notes.md`。
- `release/` 是本機交付輸出，不進版本庫；不更改上游 CI 原有的多平台產物命名。
- 複製 APK 前確認來源是本次建置產物；不得任意挑選第一個 APK，也不得為了讓版本一致只改資料夾名稱。

## 本機 Android 測試語法

以下從 repo 根目錄執行；先用 `adb devices -l` 確認目標裝置，不將歷史序號當成永遠有效的裝置。

```powershell
$version = (Get-Content "apps/readest-app/package.json" -Raw | ConvertFrom-Json).version
$apkPath = Join-Path (Get-Location) "release/$version/release.apk"
if (!(Test-Path -LiteralPath $apkPath)) { throw "找不到本次 release APK：$apkPath" }
$adbPath = "D:\platform-tools\adb.exe"
& $adbPath devices -l
# 將序號替換為本次明確選定的裝置，再執行安裝；不可直接照抄佔位字串。
& $adbPath -s "<本次裝置序號>" install -r $apkPath
```

檢查 APK metadata（`aapt`、`apksigner` 需已加入 PATH，或改用 Android SDK build-tools 的完整路徑）：

```powershell
$version = (Get-Content "apps/readest-app/package.json" -Raw | ConvertFrom-Json).version
$apkPath = Join-Path (Get-Location) "release/$version/release.apk"
aapt dump badging $apkPath
apksigner verify --verbose --print-certs $apkPath
```

比對實際 APK 版本、applicationId 與預期簽章憑證；簽章不同不得改用移除 App 的方式強行安裝，以免刪除書庫與閱讀資料。

## Git Ignore 規則

以下本機內容不得提交；檢查根目錄與子專案的 `.gitignore` 是否已涵蓋，缺漏時按實際路徑補上，不只依賴本檔文字：

- `.gradle/`、`build/`、`*/build/`、`target/`
- `node_modules/`、`.next/`、`.open-next/`、`out/`、coverage 與測試輸出
- `.idea/`、`*.iml`、`captures/`
- `local.properties`、`keystore.properties`、本機簽章腳本與私密環境設定
- `release/`、`*.apk`、`*.aab`、`*.keystore`、`*.jks`
- `graft/` 本機索引快取

本專案部分 Android 原生檔案雖位於 `src-tauri/gen/android/` 仍已受版本管理；不要因目錄叫 gen 就刪除或取消追蹤。既有依賴鎖定檔、資料庫 schema 與 migration 應依專案慣例納入版本管理，schema 修改必須和 migration 一起檢查。

<!-- graft:start -->
## Graft — repo context graph

This repo is indexed in `graft/`: small linked markdown nodes that explain each
system and carry exact file:line spans, kept in sync with the code through git.

For ANY task here — understanding how something works, finding where code lives,
or scoping a change — get context from the graph before grepping or opening
source files. Re-ask freely (it's cheap) and reuse literal identifiers you
already have (symbol, error string, file name) as the query. New to this repo?
Run `graft map` first — a token-budgeted orientation (dir clusters, hubs,
hotspots), no LLM, no key.

- Run `graft ask "<your question>" --source` → ranked nodes with the relevant
  code spans inlined (each hit's ≤8-line crux by default; `--full` for whole
  definitions when the crux isn't enough). Match the tool to the task shape:
  for understanding or editing, the top node IS the answer — cite its
  `covers:` file:line spans and edit straight from `--source`. For
  exhaustive tasks ("every occurrence / every caller of this pattern"), ranked
  results are top-N, not complete — run `graft grep "<literal>"` instead
  (exhaustive over indexed files, grouped by enclosing symbol), falling back
  to raw `grep -rn` only for unindexed files.
- `graft skeleton <file>` → every definition's signature + span, ~10× cheaper
  than reading the file; use it to skim an API surface.
- `graft callers <symbol>` gives precomputed, exact edges — who calls this.
  Add `--direction out` for what it calls, or `--depth N` to walk
  transitively for the full blast radius. For structural questions, skip
  ranking and use this directly.
- Or browse: `graft/INDEX.md` lists every node; follow the links.
- Monorepos and folders of multiple repos rank fairly across sub-projects —
  hits carry `[scope/]` labels naming which one they're from. Narrow with
  `graft ask "<task>" --in <scope>/` once you know where you're working.

If a returned span is truncated ("+N more lines"), open the file at that exact
range before finalizing. Only open source files when a node genuinely lacks a
needed detail, and then at the exact file:line the node points to — never
re-read whole files.

After big code changes, refresh the graph with `graft build` (deterministic,
no API key, $0).
<!-- graft:end -->
