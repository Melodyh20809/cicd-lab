# CI Pipeline 報告（HW4）

## 1. CI Pipeline 說明

本作業使用 GitHub Actions 建立 CI pipeline，當專案發生 `push` 時會自動觸發。流程包含：

1. 安裝 Node.js 與相依套件
2. 執行 TypeScript 型別檢查（`npm run typecheck`）
3. 執行 Prettier 格式檢查（`npm run format:check`）
4. 執行測試（`npm run test`）並輸出 JUnit 測試結果
5. 將測試結果上傳並顯示在 GitHub Actions 結果頁面

只要任一步驟失敗，整個 pipeline 會失敗。

## 2. workflow 主要內容

檔案：`.github/workflows/ci_學號.yaml`

```yaml
name: CI

on:
  push:

jobs:
  validate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      checks: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: TypeScript typecheck
        run: npm run typecheck

      - name: Prettier check
        run: npm run format:check

      - name: Run tests with JUnit output
        run: npm run test -- --reporter=default --reporter=junit --outputFile=./test-results/vitest-junit.xml

      - name: Upload test report artifact
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/vitest-junit.xml
          if-no-files-found: error

      - name: Publish test result summary
        if: always()
        uses: dorny/test-reporter@v1
        with:
          name: Vitest Results
          path: test-results/vitest-junit.xml
          reporter: java-junit
```

## 3. 設計策略說明

- 將流程拆為三個獨立 job（TypeScript Typecheck、Prettier Check、Test），在 GitHub Actions 首頁可直接看出是哪個檢查失敗。
- 透過 JUnit 輸出 + `dorny/test-reporter`，讓測試結果在 GitHub Actions 頁面可視化顯示。
- `if: always()` 讓測試摘要上傳步驟即使前面失敗也盡量保留資訊，方便除錯。

## 4. CI 執行結果截圖（成功）

請貼上至少一張成功截圖：

- GitHub Actions workflow run 頁面（狀態為綠色 Success）
- 可加上 Vitest Results summary 畫面

## 5. 失敗案例說明（範例）

### 錯誤製造

示範：在某個 `.ts` 檔案故意寫入型別錯誤，例如把 `number` 指派成 `string`。

### 錯誤結果

- pipeline 在 `TypeScript typecheck` 步驟失敗
- GitHub Actions 顯示紅色 Failed

### 修正方式

- 將錯誤型別改正後重新 push
- pipeline 回到 Success

（你也可以改用 Prettier 格式錯誤或測試失敗作為失敗案例。）

## 6. 使用工具

- GitHub Actions
- `actions/checkout@v4`
- `actions/setup-node@v4`
- `actions/upload-artifact@v4`
- `dorny/test-reporter@v1`
- Vitest（JUnit reporter）
