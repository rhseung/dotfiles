# dotfiles

[chezmoi](https://www.chezmoi.io/) 로 관리하는 macOS 개발 환경 설정.
만든 과정은 [**개발자 Macbook 종합 세팅**](https://www.rhseung.me/ko/blog/mac-settings/) 에 있다.

## Prerequisites

- **Homebrew**
- **1Password** 앱 + 설정 > 개발자 > "1Password CLI 와 통합"
- **App Store 로그인** — 안 하면 Brewfile 의 `mas` 줄이 조용히 실패한다.
  macOS 10.13 부터 `mas signin` 이 막혀 App Store 앱에서 직접 로그인해야 한다.
- 언어 런타임: [mise](https://mise.jdx.dev/), [uv](https://docs.astral.sh/uv/), [Bun](https://bun.com/)

## Bootstrap

```sh
# chezmoi 없으면
brew install chezmoi   # 또는: sh -c "$(curl -fsLS get.chezmoi.io)"

chezmoi init --apply https://github.com/rhseung/dotfiles.git
brew bundle --file ~/.local/share/chezmoi/Brewfile
```

`chezmoi apply` 가 dotfiles 를 풀고 `run_onchange_` 스크립트까지 돌린다.
Brewfile 은 chezmoi 가 관리하지 않아서 `brew bundle` 은 따로 실행한다.

## Managed Files

| 소스 | 대상 | 비고 |
| --- | --- | --- |
| `dot_zshrc` | `~/.zshrc` | PATH → completions → 플러그인(순서 고정) → 툴 init → 프롬프트. 툴 없으면 건너뜀 |
| `dot_zprofile` | `~/.zprofile` | mise `--shims` (GUI·비인터랙티브 셸용) |
| `private_dot_gitconfig` | `~/.gitconfig` | delta 페이저, zdiff3, SSH 커밋 서명(1Password) |
| `dot_config/git/ignore` | `~/.config/git/ignore` | 전역 gitignore |
| `dot_config/ghostty/config` | `~/.config/ghostty/config` | 폰트·테마·키바인드 |
| `dot_config/starship.toml` | `~/.config/starship.toml` | 프롬프트 |
| `private_Library/LaunchAgents/local.hidutil.rcmd-to-f18.plist` | `~/Library/LaunchAgents/…` | 오른쪽 Cmd → F18 리매핑 |
| `private_Library/…/Code/User/settings.json.tmpl` | VS Code `settings.json` | Flow Icons 라이선스를 1Password 에서 읽는다 |

## Repo Assets

chezmoi 가 관리하지 않는다 (`.chezmoiignore`).

| 파일 | 용도 |
| --- | --- |
| `Brewfile` | `brew bundle` 입력. 블로그 글에서 깐 formula/cask/mas 만 |
| `vscode-extensions.txt` | VS Code 확장 ID 목록. 빈 줄로 그룹 구분 |
| `uv-tools.txt` | `uv tool` 로 까는 것 목록. brew 에 formula 가 없는 것만 |

## Run Scripts

`chezmoi apply` 중 실행. `run_onchange_` 는 스크립트 본문(해시 주석 포함)이 바뀔 때만 다시 돈다.

| 스크립트 | 하는 일 |
| --- | --- |
| `run_onchange_after_bootstrap-hidutil.sh.tmpl` | plist 가 바뀌면 LaunchAgent `launchctl bootout` + `bootstrap` |
| `run_onchange_after_install-vscode-extensions.sh.tmpl` | 목록이 바뀌면 `code --install-extension` 루프 |
| `run_onchange_after_install-uv-tools.sh.tmpl` | 목록이 바뀌면 `uv tool install --upgrade` 루프 |

## Secrets

- 토큰·비번은 저장소에 없다. `fnox` 또는 1Password 로 환경에 주입한다.
- Flow Icons 라이선스만 예외로, `settings.json.tmpl` 의
  `{{ onepasswordRead "op://Private/Flow Icons/reg_code" }}` 로 apply 할 때 채운다.
- `private_` 접두가 붙은 것(`~/.gitconfig`, LaunchAgent, VS Code 설정)은 600 권한으로 풀린다.

## Not Included

- 언어 런타임(mise/uv/bun) - 셋 다 자체 업데이트 명령을 살리려고 설치 스크립트로 깐다
- Raycast(직접 설치), MS Office·한컴(교내 배포처), MonoLisa(유료 폰트)
- VS Code 설정 동기화는 Settings Sync 와 병행 — 확장 목록만 여기 둔다
