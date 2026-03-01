# 📄 技术实战指南：如何在在线 IDE 中突破限制加载超大 PDF 文件

## 1. 背景与痛点
在开发基于 React/Vue 的在线作品集或文档查看器时，常遇到以下限制：
- **文件大小限制**：在线 IDE（如 Luma, StackBlitz, CodeSandbox）或云存储通常限制单个上传文件不超过 25MB~30MB。
- **内存限制**：浏览器端直接渲染超过 50MB 的 PDF 极易导致页面崩溃。
- **需求**：用户有一个接近 100MB 的完整简历/作品集 PDF，需要在前端实现“无缝浏览”，且看起来像一个完整文件。

## 2. 核心解决方案
采用 **“本地拆分 + GitHub 托管 + 前端拼接”** 的策略：
1.  **拆分**：将大 PDF 本地拆分为多个小于 25MB 的小文件。
2.  **托管**：利用 GitHub 仓库作为免费、高速、支持大文件（单文件<100MB）的静态资源服务器。
3.  **拼接**：前端通过 `react-pdf` 库依次加载这些文件的公开链接（Raw URL），并模拟连续翻页效果。

---

## 3. 详细操作步骤

### 第一步：准备文件
使用本地工具（如 Adobe Acrobat, Python `PyPDF2`, 或在线拆分工具）将原始大文件拆分为小文件：
- `part1.pdf` (约 30MB)
- `part2.pdf` (约 30MB)
- `part3.pdf` (约 30MB)

### 第二步：创建 GitHub 仓库
1.  登录 GitHub，点击 **New Repository**。
2.  **Repository name**: 例如 `my-resume-assets`。
3.  **Visibility**: 必须选择 **Public** (公开)，否则无法获取公开访问链接。
4.  **Initialize**: 勾选 "Add a README file" 和 "Choose a license" (推荐 MIT)。
5.  点击 **Create repository**。

### 第三步：使用 Git 命令行上传大文件
*注意：GitHub 网页端拖拽上传限制为 25MB，必须使用命令行上传（支持最大 100MB）。*

1.  **克隆仓库到本地**：
    ```bash
    git clone https://github.com/你的用户名/你的仓库名.git
    cd 你的仓库名
    ```

2.  **放入文件**：
    将拆分好的 `part1.pdf`, `part2.pdf`, `part3.pdf` 复制到该文件夹内。

3.  **提交并推送**：
    ```bash
    # 添加文件
    git add .

    # 提交更改
    git commit -m "Add split PDF parts"

    # (可选) 如果文件较大导致推送失败，先执行以下命令增大缓冲区
    git config --global http.postBuffer 524288000

    # 推送到 GitHub
    git push origin main
    ```

### 第四步：获取 Raw 链接
1.  在 GitHub 仓库页面点击已上传的 PDF 文件（如 `part1.pdf`）。
2.  点击右上角的 **Raw** 按钮。
3.  复制浏览器地址栏的 URL。
   - ✅ 正确格式：`https://raw.githubusercontent.com/用户名/仓库名/main/part1.pdf`
   - ❌ 错误格式：`https://github.com/用户名/仓库名/blob/main/part1.pdf` (这是预览页，不能用于代码加载)

对每个分片文件重复此操作，得到一组链接数组。

---

## 4. 前端代码实现 (React + react-pdf)

安装依赖：
```bash
npm install react-pdf
