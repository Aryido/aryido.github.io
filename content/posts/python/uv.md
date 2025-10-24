---
title: "uv - Python package and project manager"

author: Aryido

date: 2025-10-04T21:52:30+08:00

thumbnailImage: "/images/python/python-logo.jpg"

categories:
  - language
  - python

comment: false

reward: false
---

<!--BODY-->
> uv 是 Astral 公司推出的一款基於 Rust 編寫的「 **Python 套件管理工具** 」，雖然 Python 的套件管理生態內已經存在多種工具如 pip 、 poetry 、 conda 等等，但在效能、相容性和功能上 uv 都有出色表現，得益於使用 Rust 進行編寫，uv 工具的速度讓人驚艷 ；且從「 安裝管理 Python 版本 」、「 虛擬環境建置」、「 package 依賴 」以上 uv 全部都統整好了，其目標是取代多種現有工具，為 Python 專案的開發和管理帶來了新的選擇。 uv 也成為 Astral 公司繼 Ruff (Python 程式碼檢查器和格式化工具) 之後，另一個知名的專案了。

<!--more-->

---

{{< image classes="fancybox fig-100" src="/images/python/uv/uv-feature.jpg" >}}


一直以來 python 管理開發環境的的各式工具都混亂到讓人頭疼，多種工具分散使用如：
- 下載 python 最新版本，有人去官網，有人用 homebrew、apt、yum 或者 pyenv ...
- 為了避免依賴衝突，使用 「venv」或者 「virtual env」建立虛擬環境
- 安裝 package 方式也非常多種： pip 、 poetry ...

那 uv 目前集成以上所有的功能，以下筆記使用 uv 開發的常用 CLI。

# uv 管理 python 版本

uv 可以安裝和管理不同的 Python 版本，

```bash
# 列出 uv 支援的 python 版本
uv python list

#  安裝某個 python 版本
uv python install XXXX7

# 列出目前已安裝的 Python 路徑
uv python dir 

# 使用特定版本 python 如 3.13 執行 xxx.py ; 如果沒安裝的話會直接幫忙安裝
uv run -p 3.13 xxx.py 
# python互動介面
uv run -p 3.13 python

# 使用系統 python 或目前工程的虛擬環境執行 xxx.py
uv run xxx.py
```

---

# 使用 uv 來管理 Python 專案

```bash
# 當前目錄創建 uv 工程 ，並使用 python 3.13
uv init -p 3.13

# 選定 python 版本，指定創建專案的目錄
uv python pin 3.11
uv init <PROJECT_DIR>

uv init <PROJECT_DIR> --python 3.13
```

透過 init 建立專案之後，會**自動將專案使用 Git 來管理**，且 uv 工具會產生了一些預設檔，而 uv 主要是有兩個 file：

- **pyproject.toml** ：定義專案的主要依賴，包括專案名稱、版本、支援的 Python 版本等訊息
- **uv.lock** ：記錄項目的所有依賴，包括依賴的依賴，且跨平台，確保在不同環境下安裝的一致性
  {{< alert warning >}}
  `uv.lock` 這個檔案就是專案依賴關係的完整快照，它能確保專案在不同機器上的運作環境保持一致，詳細記錄了每個套件以及它依賴的套件的情況，由 uv 自動管理，不要手動編輯
  {{< /alert >}}

其他還有：
- **.python-version** : 這個專案使用的 Python 版本，如果在 uv init 的時候沒有指定 Python 版本，那麼就會用已安裝的最高版本。

```bash
# 同步專案依賴
# 同步會自動尋找或下載適當的 Python 版本
#   建立並設定專案的虛擬環境
#   建立完整的依賴清單並寫入 uv.lock 文件
#   最後將依賴同步到虛擬環境
uv sync

# 新增依賴 ; 若沒有虛擬環境，這時 uv 也會順便幫忙創建 .venv
# 會修改 pyproject.toml 與 uv.lock
uv add XXX

# 刪除依賴 
uv remove XXX

# 列印依賴樹
uv tree

# 有使用 uv 管理專案的話，可以用 uv 的命令來運行程式碼，不要像以前那樣使用 python xxx.py 來運行
uv run XXX.py
# 指定 Python 3.11 版本來執行
uv run --python 3.11 XXX.py

# 目錄下生成虛擬環境
uv venv


# 管理臨時的虛擬環境
uv cache dir 
uv cache clean 

```

---

# uv tool 

uv tool 會在一個臨時的、隔離的環境中安裝和執行 Python 工具。 例如想用 Ruff 工具來檢查一下 code 是否符合規範，但是我們現在的環境中並沒有安裝 Ruff。解決這個問題最直接的是把想用的工具當作依賴引入進來 :
```
uv add ruff --dev
```

在這裡特別加上一個 `--dev` 是避免在打包的時候把 Ruff 也一起打包進來，但其實使用了 `--dev` 也不太合適，因為它本身是一個工具和 code 是不相關的，適合使用 `--dev` 的範例比如說:
- 單元測試用到的 `pytest`
- mock lib

故比較推薦使用 `uv tool install ruff`，這時候 Ruff 就是脫離目前的專案獨立運作的，可以使用 `which` 來查看到它的路徑並不是當前的虛擬環境

{{< alert info >}}
用 `uv tool install` 安裝的工具是整個系統都可用的，而且 uv 在安裝工具的時候會為每個工具建立自己的虛擬環境，所以不需要擔心 lib 之間的衝突
{{< /alert >}}

```bash
uv tool list

uv tool install <PACKAGE NAME>
uv tool upgrade <PACKAGE NAME>
uv tool uninstall <PACKAGE NAME>


### 以下範例使用 uv 配合 poetry 管理的專案 ###

# uv 管理 poetry
uv tool install poetry

# 使用 uv 管理的 poetry 來安裝專案依賴
uv tool run poetry install

```

---

### 參考資料

- [uv](https://github.com/astral-sh/uv)
- [用uv管理Python的一切！](https://www.youtube.com/watch?v=aVXs8lb7i9U)
- [uv：統一的 Python 包管理](https://www.readfog.com/a/1767351325973123072)
- [Python包管理不再头疼：uv工具快速上手](https://www.cnblogs.com/wang_yb/p/18635441)
- [uv全功能更新：统一管理Python项目、工具、脚本和环境的终极解决方案](https://segmentfault.com/a/1190000046519293)
- [UV：基於 Rust 的超高速 Python 包管理工具](https://calpa.me/blog/uv-rust-python-package-manager/)
- [uv 簡單介紹](https://blog.awin.one/posts/2025/uv-%E7%B0%A1%E5%96%AE%E4%BB%8B%E7%B4%B9/)
