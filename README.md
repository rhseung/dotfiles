# dotfiles

[chezmoi](https://www.chezmoi.io/)로 관리하는 macOS 개발 환경 설정입니다.
구성 과정은 [개발자 Macbook 종합 세팅](https://www.rhseung.me/ko/blog/mac-settings/)에
정리해 두었습니다.

## 사전 준비

- Homebrew를 설치합니다.
- 1Password 앱을 설치하고, 설정 > 개발자 > "1Password CLI와 통합" 을 켭니다.
- App Store에 로그인합니다. 로그인하지 않으면 Brewfile의 `mas` 줄이 아무 메시지 없이 실패합니다. macOS 10.13부터 `mas
signin`이 막혔기 때문에 App Store 앱에서 직접 로그인해야 합니다.
- 언어 런타임으로 [mise](https://mise.jdx.dev/), [uv](https://docs.astral.sh/uv/), [Bun](https://bun.com/)을 사용합니다.

## 설치

```sh
# chezmoi가 없으면
brew install chezmoi   # 또는: sh -c "$(curl -fsLS get.chezmoi.io)"

chezmoi init --apply https://github.com/rhseung/dotfiles.git
brew bundle --file ~/.local/share/chezmoi/Brewfile
```

`chezmoi apply`가 dotfiles를 배치하고 `run_onchange_` 스크립트까지 실행합니다.
Brewfile은 chezmoi가 관리하지 않으므로 `brew bundle`은 따로 실행합니다.

## 관리 대상 파일

| 소스 | 대상 | 비고 |
| --- | --- | --- |
| `dot_zshrc` | `~/.zshrc` | PATH, completions, plugin, tool init, prompt 순서로 고정. 툴이 없으면 건너뜁니다 |
| `dot_zprofile` | `~/.zprofile` | mise `--shims` (GUI 및 비인터랙티브 셸용) |
| `private_dot_gitconfig` | `~/.gitconfig` | delta pager, zdiff3, SSH 커밋 서명 (1Password) |
| `dot_config/git/ignore` | `~/.config/git/ignore` | 전역 gitignore |
| `dot_config/ghostty/config` | `~/.config/ghostty/config` | font, theme, keybind |
| `dot_config/starship.toml` | `~/.config/starship.toml` | prompt |
| `private_Library/LaunchAgents/local.hidutil.rcmd-to-f18.plist` | `~/Library/LaunchAgents/...` | 오른쪽 Cmd 키를 F18로 리매핑 |
| `private_Library/LaunchAgents/local.update-all.plist.tmpl` | `~/Library/LaunchAgents/...` | 매일 13:00에 `update-all` 실행 |
| `dot_local/bin/executable_update-all` | `~/.local/bin/update-all` | brew, mise, uv 일괄 업데이트 |
| `private_Library/.../Code/User/settings.json.tmpl` | VS Code `settings.json` | Flow Icons 라이선스를 1Password에서 읽습니다 |

## 저장소 자산

chezmoi가 관리하지 않습니다 (`.chezmoiignore`).

| 파일 | 용도 |
| --- | --- |
| `Brewfile` | `brew bundle`의 입력. 블로그 글에서 설치한 formula, cask, mas만 기재합니다 |
| `vscode-extensions.txt` | VS Code 확장 ID 목록. 빈 줄로 그룹을 구분합니다 |
| `uv-tools.txt` | `uv tool`로 설치하는 목록. brew에 formula가 없는 것만 기재합니다 |

## 실행 스크립트

`chezmoi apply` 도중에 실행됩니다. `run_onchange_`는 스크립트 본문 (해시 주석 포함) 이
바뀔 때만 다시 실행됩니다.

| 스크립트 | 하는 일 |
| --- | --- |
| `run_onchange_after_bootstrap-launchagents.sh.tmpl` | plist가 바뀌면 LaunchAgent를 `launchctl bootout` 한 뒤에 `bootstrap` 합니다 |
| `run_onchange_after_install-vscode-extensions.sh.tmpl` | 목록이 바뀌면 `code --install-extension`을 반복 실행합니다 |
| `run_onchange_after_install-uv-tools.sh.tmpl` | 목록이 바뀌면 `uv tool install --upgrade`를 반복 실행합니다 |

## 자동 업데이트

`local.update-all` LaunchAgent가 매일 13:00에 `~/.local/bin/update-all`을 실행합니다.
컴퓨터가 잠자는 동안 지나간 시각은 깨어날 때 한 번 몰아서 실행됩니다.

| 대상 | 명령 |
| --- | --- |
| Homebrew | `brew update`, `brew upgrade --formula`, 그다음 cask를 하나씩 |
| mise 자체 | `mise self-update -y` |
| mise 글로벌 툴 | `mise -C "$HOME" upgrade`. 홈에 `mise.toml`이 없으므로 글로벌 config의 항목만 대상이 됩니다 |
| uv tool | `uv tool upgrade --all` |

`update-all`은 launchd가 넘겨준 환경을 신뢰하지 않고 PATH를 직접 설정합니다. 이때
`export`가 `brew shellenv`보다 먼저 실행되어야 합니다. `brew shellenv` 안의
`path_helper`가 환경변수 PATH만 읽으므로, export 되지 않은 PATH로는 `/usr/bin`이
통째로 사라지기 때문입니다. 한 단계가 실패해도 나머지 단계는 실행되도록 `set -e`는 사용하지
않습니다.

로그는 `~/Library/Logs/update-all.log`에 기록됩니다. 지금 바로 실행하려면 `update-all`
또는 `launchctl kickstart -k gui/$(id -u)/local.update-all`을 실행합니다.

cask는 `--greedy`로 목록만 뽑은 뒤에 하나씩 업그레이드합니다. `brew info --json=v2`에
`pkgutil`, `launchctl`, `kext`가 포함된 cask (zoom 등) 는 업그레이드할 때 sudo가
필요한데, launchd에는 TTY가 없으므로 `sudo: a terminal is required` 오류로 종료됩니다.
그래서 그런 cask는 건너뛰고 `skip cask <이름> - sudo 필요` 만 로그에 남깁니다. 해당
cask는 `brew upgrade --cask <이름>` 으로 직접 업그레이드합니다.

## 비밀 정보

- 토큰과 비밀번호는 저장소에 두지 않습니다. `fnox` 또는 1Password로 환경에 주입합니다.
- Flow Icons 라이선스만 예외입니다. `settings.json.tmpl`의 `{{ onepasswordRead "op://Private/Flow Icons/reg_code" }}` 로
apply 할 때 값을 채웁니다.
- `private_` 접두가 붙은 파일 (`~/.gitconfig`, LaunchAgent, VS Code 설정) 은 600 권한으로 배치됩니다.

## 제외 대상

- 언어 런타임 (mise, uv, bun). 세 가지 모두 자체 업데이트 명령을 유지하기 위해 설치 스크립트로 설치합니다.
- Raycast (직접 설치), MS Office와 한컴 (교내 배포처), MonoLisa (유료 폰트)
- VS Code 설정 동기화는 Settings Sync와 병행합니다. 확장 목록만 이 저장소에 둡니다.
