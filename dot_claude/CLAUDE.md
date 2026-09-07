# Global Claude Rules

## CLI Tool Preferences

Use modern CLI alternatives — these are installed and aliased in zshrc:

| Task | Use | Instead of |
|---|---|---|
| List files | `eza` | `ls` |
| Find files | `fd` or `fzf` | `find` |
| Search text | `rg` (ripgrep) | `grep` |
| Edit files | `micro` | `nano`, `vi`, `vim` |

Reading files: use the Read tool, not `cat`/`bat`. `bat`'s pager, line numbers, and syntax colors are for humans, not agents.

Parse JSON with `jq` and YAML/TOML with `yq` rather than slicing them with grep/sed.

Drive the browser with `agent-browser`, not the Playwright MCP.

When running shell commands, always prefer these tools. They are faster and installed globally.

## Commits

**커밋 하나에 변경 하나.** 판정은 `git diff --staged` 로 한다 - 스테이지한 변경이 서로
독립적으로 되돌려질 수 있으면 커밋 두 개다. 리팩터와 기능, 서식과 로직을 같이 담지 않는다.
한 덩어리가 아니면 `git add -p` 로 나눠 담는다.

이건 **지금 있는 변경을 어떤 기준으로 쪼갤지** 에 대한 규칙이다. 한 작업의 커밋 수를 1 개로
유지하라는 뜻이 아니다.

**이미 만든 커밋은 건드리지 않는다.** 리뷰 지적이든 추가 수정이든 새 커밋으로 쌓는다.
커밋은 무엇이 언제 왜 바뀌었는지의 기록이라, 갈아엎으면 그 기록이 "처음부터 맞게 썼다" 는
거짓이 된다. `--amend` `--fixup` `rebase` 로 히스토리를 다시 쓰는 건 명시적으로 요청받았을
때만 한다. 리뷰가 열려 있는 브랜치에는 force-push 하지 않는다 - 리뷰 스레드가 끊긴다.

**제목은 영어, 본문은 한국어.** 제목은 `git log --oneline` 과 GitHub 목록에 서고 grep 대상이다.
영어로 옮기면 뜻이 흐려지는 고유명사와 도메인 용어만 한국어로 남긴다.
왜 그렇게 했는지는 본문에 한국어로 쓴다.

**문장부호는 ASCII 만.** `·` `—` `→` `…` 같은 기호는 터미널마다 폭이 달라 정렬이 깨진다.

프로젝트에 커밋 타입 규약이 따로 없으면 `<type>: <title>`, 명령형으로 쓴다.

## Comments

**주석은 코드가 말할 수 없는 것만 적는다.** 무엇을 하는지는 코드가 이미 말한다 -
적을 값어치가 있는 건 *왜 이 방식이어야 하는지*, 특히 **다르게 짜면 깨지는 이유**다.
함수·타입·prop 에 이름값을 되풀이하는 설명형 JSDoc 을 붙이지 않는다.
