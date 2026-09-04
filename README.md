# dotfiles

[chezmoi](https://www.chezmoi.io/) 로 관리하는 macOS 개발 환경 설정.
이 저장소가 어떻게 만들어졌는지는 블로그 글 **개발자 Macbook 종합 세팅** 에 있다.

## 새 맥 부트스트랩

```sh
# chezmoi 없으면
brew install chezmoi   # 또는: sh -c "$(curl -fsLS get.chezmoi.io)"

chezmoi init --apply https://github.com/rhseung/dotfiles.git
brew bundle --file ~/.local/share/chezmoi/Brewfile
```

`chezmoi apply` 가 dotfiles 를 풀고 `run_onchange_` 스크립트(아래)까지 돌린다.
`brew bundle` 은 따로 실행한다 — Brewfile 은 chezmoi 가 관리하지 않는 저장소 자산이다.

선행 조건:

- **Homebrew**
- **1Password** 앱 + 설정 > 개발자 > "1Password CLI 와 통합"
- 언어 런타임([mise](https://mise.jdx.dev/)), [uv](https://docs.astral.sh/uv/), [Bun](https://bun.com/)

## 담긴 것

| 소스 | 대상 | 비고 |
| --- | --- | --- |
| `dot_zshrc` | `~/.zshrc` | PATH → completions → 플러그인(순서 고정) → 툴 init → 프롬프트. 툴 없으면 조용히 건너뜀 |
| `dot_zprofile` | `~/.zprofile` | mise `--shims` (GUI·비인터랙티브 셸용). OrbStack·JetBrains 줄은 걔네가 재설치 때 다시 붙임 |
| `private_dot_gitconfig` | `~/.gitconfig` | delta 페이저, zdiff3, SSH 커밋 서명(1Password). `signingkey` 는 공개키 |
| `dot_config/git/ignore` | `~/.config/git/ignore` | 전역 gitignore |
| `dot_config/ghostty/config` | `~/.config/ghostty/config` | 폰트·테마·키바인드 |
| `dot_config/starship.toml` | `~/.config/starship.toml` | 프롬프트 |
| `private_Library/LaunchAgents/local.hidutil.rcmd-to-f18.plist` | `~/Library/LaunchAgents/…` | 오른쪽 Cmd → F18 리매핑 |
| `private_Library/…/Code/User/settings.json.tmpl` | VS Code `settings.json` | `.tmpl` — Flow Icons 라이선스를 1Password 에서 읽는다 |

## 저장소 자산 (chezmoi 관리 안 함, `.chezmoiignore`)

| 파일 | 용도 |
| --- | --- |
| `Brewfile` | `brew bundle` 입력. 블로그 글에서 깐 formula/cask/mas 만 |
| `vscode-extensions.txt` | 설치할 VS Code 확장 ID 목록. 빈 줄로 그룹 구분, 주석 없음 |
| `README.md` | 이 파일 |

## run 스크립트

`chezmoi apply` 중 실행된다. `run_onchange_` 는 스크립트 본문(해시 주석 포함)이 바뀔 때만 다시 돈다.

| 스크립트 | 하는 일 |
| --- | --- |
| `run_onchange_after_bootstrap-hidutil.sh.tmpl` | plist 가 바뀌면 LaunchAgent 를 `launchctl bootout` + `bootstrap` |
| `run_onchange_after_install-vscode-extensions.sh.tmpl` | `vscode-extensions.txt` 가 바뀌면 `code --install-extension` 루프 |

## 시크릿

- **시크릿 값**(토큰·비번) 은 저장소에 없다. `fnox` 또는 1Password 로 환경에 주입한다.
- Flow Icons 라이선스만 예외적으로 필요한데, 평문 대신 `settings.json.tmpl` 의
  `{{ onepasswordRead "op://Private/Flow Icons/reg_code" }}` 로 apply 때 채운다.
- `~/.gitconfig`, LaunchAgent, VS Code 설정은 `private_` 접두 — 600 권한으로 풀린다.

## 안 담긴 것

- 언어 런타임(mise/uv/bun), `uv tool` 로 까는 것(ruff·pyrefly·claude-swap)
- Raycast(직접 설치), MS Office·한컴(교내 배포처), MonoLisa(유료 폰트)
- VS Code 확장은 목록(`vscode-extensions.txt`)만 두고 설정 동기화는 Settings Sync 와 병행
