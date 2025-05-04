name: Auto Update _worker.js

on:
  schedule:
    - cron: '0 0 * * *'   # 每天00:00 UTC运行
  workflow_dispatch:      # 允许手动触发

jobs:
  update:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout 仓库
        uses: actions/checkout@v4

      - name: 获取最新 release 信息
        id: get_release
        run: |
          latest_tag=$(curl -s https://api.github.com/repos/bia-pain-bache/BPB-Worker-Panel/releases/latest | jq -r '.tag_name')
          echo "最新版本: $latest_tag"
          echo "latest_tag=$latest_tag" >> $GITHUB_ENV

      - name: 检查本地 version.txt
        id: check_version
        run: |
          if [ -f "version.txt" ]; then
            current_version=$(cat version.txt | tr -d '\n' | tr -d '\r')
            echo "当前本地版本: $current_version"
          else
            echo "未找到 version.txt，视为首次更新。"
            current_version=""
          fi
          echo "current_version=$current_version" >> $GITHUB_ENV

      - name: 判断是否需要更新
        id: need_update
        run: |
          if [ "$current_version" = "$latest_tag" ]; then
            echo "No update needed."
            echo "need_update=false" >> $GITHUB_ENV
          else
            echo "Update needed."
            echo "need_update=true" >> $GITHUB_ENV
          fi

      - name: 下载并替换 _worker.js
        if: env.need_update == 'true'
        run: |
          download_url="https://github.com/bia-pain-bache/BPB-Worker-Panel/releases/download/${{ env.latest_tag }}/worker.js"
          echo "下载文件: $download_url"
          curl -L -o _worker.js "$download_url"
          echo "${{ env.latest_tag }}" > version.txt

      - name: 提交更新到 GitHub
        if: env.need_update == 'true'
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add _worker.js version.txt
          git commit -m "Update _worker.js to version ${{ env.latest_tag }}"
          git push
