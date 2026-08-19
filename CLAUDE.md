# BatchForNewProduct - Claude Code 開發規範

## Git Repository
Repository:
https://github.com/chimoon168/BatchForNewProduct
Default Branch:
main

## Git Workflow
修改程式前：
1. 先執行 `git status`
2. 確認目前所在 branch
3. 修改前不要執行 push

完成修改後：
1. 執行 `git diff`
2. 確認修改內容
3. 執行相關測試
4. 執行 `git status`
5. 建立 Git commit
6. Push 到目前工作 branch

## Push 規則
除非我明確要求，否則：
- 不要直接 push 到 main
- 不要執行 force push
- 不要使用 `git push --force`
- 不要修改 remote URL
- 不要修改 Git Credential Manager 設定
- 不要要求我提供 GitHub Password
- 不要要求我提供 GitHub Personal Access Token

GitHub Authentication 已經由 Windows Git Credential Manager 管理。
目前 Git Credential Helper：
`manager`

## Commit Message
Commit message 使用簡潔的英文描述，例如：
- Add product batch submission validation
- Fix draft approval page
- Update product submission UI
- Fix product code validation

## Deploy / Push
當我說：
「推版」
代表：
1. 檢查 git status
2. 檢查 git diff
3. 確認修改內容
4. 執行必要測試
5. 建立 commit
6. 使用現有 Git Credential Manager 執行 git push

如果發現：
- merge conflict
- authentication error
- permission error
- branch 不正確
- remote 不正確

請停止，不要自行修改 Git 設定，並告訴我錯誤原因。
