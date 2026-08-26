autolabのプロジェクトワークスペースの自由度を向上させたい。
今の固定的なリポジトリ構成から、autolabエージェントの判断と非決定論的ワークフローでプロジェクトに応じたリポジトリ構成で作業できるようにした上で、パターン化する。

まず以下のようなworkplan/workrunのguideの中の記述はガイドから削除した。
The folder "main/" contains current source codes of the project.
The folder "direction/", if exists, contains concept documents of the project.
The folder "devlog/", if exists, contains plans and reports of past works.
Being empty means the project has just started. 

そして以下で置き換えた。
The file "README_PROJECT.md" explains how those project folders are supposed to work.

workplanのガイドに以下も加えた
You edit those files only when you added new repositories or local folders in the workspace, or changed the way to manage development of the project.

フォルダの説明はREADME_PROJECT.mdの方に移す。これはエージェント自身に作らせる方針にする。以下の文章を加えておいた。
if "README_PROJECT.md" dosn't exist, create it to explain how each folders works.

なお、giteaリポジトリの作成はautolabコマンドでできるようにする。


またautolab doc patternsというコマンドをworkplanで使えるようにする。
pj-agdev/agautolab/agent/project_pattern.mdを返すだけ。

The file "autolab doc patterns" explains how project structure should be managed based on pattern. If you are asked to create project based on specific pattern, follow it. If not enough information is provided, or specified nonexistent pattern, just ask back in reply.

ただ、プロジェクトマネジメント関連のcliコマンドはすでにあった気がするので、もしすでにあればそっちのコマンド書式に自然に合わせる形でいい。

この状態で、新規に#pj-studyarxivチャンネルを作り、"studyパターンプロジェクトを、publishリポジトリをhttps://github.com/iwaag/study-arxiv-torend、mainリポジトリをhttp://agstudio:3000/autodev/papers.gitにして作って。"とworkplan-で依頼。想定通りプロジェクトワークスペースを作るか見てみる。

publish/フォルダーのgithubへのpushは行わず、後で人間が手動で行う前提でいい。