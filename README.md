# Vertical Shooter (S/W engineering proj 1)

화면 아래쪽 플레이어 비행기를 좌우로 움직이며 총알을 쏘아 위에서 내려오는 적을 맞히는 종스크롤 슈팅 게임입니다.

## 실행 방법

- 아래 명령어를 사용하여 Pygame 라이브러리를 설치합니다. 본 문서 갱신 시점에서 최신 버전인 Pygame 2.6.1은 Python 3.14에 대한 [사전 빌드 패키지](https://pypi.org/project/pygame/2.6.1/#files) 를 제공하지 않기 떄문에 설치 과정에서 오류가 발생할 수 있습니다.
    ```bash
    pip install pygame
    ```
- 다음 명령어를 사용하여 실행합니다.
    ```bash
    py main.py
    ```
- 여러 버전의 Python이 설치되어 있는 경우 터미널에서 다음 명령어를 사용하여 실행합니다.
    ```bash
    py -3.XX main.py
    ```

- VS Code 에디터에서 하단 상태 표시줄이나 명령 팔레트(Ctrl+Shift+P 또는 Cmd+Shift+P)에서 Python: Select Interpreter를 열고 원하는 버전을 선택한 뒤 실행합니다.
- PyCharm 에디터에서 **설정(Settings) > 프로젝트 인터프리터(Project Interpreter)** 에서 원하는 버전의 실행 파일을 인터프리터로 지정합니다.

## 조작

| 키 | 동작 |
|---|---|
| ← / → | 좌우 이동 |
| Space | 시작 화면에서 게임 시작 / 게임 중 총알 발사 |
| R | 게임 오버 후 다시 시작 |
| Esc | 종료 (시작 화면에서도) |

## 파일 구성

| 파일 | 내용 |
|---|---|
| `main.py` | 게임 루프 — 입력 처리, 갱신, 그리기, 점수·생명·게임 오버 |
| `entities.py` | 그림 불러오기(`load_frame`), 배경(`Background`), 스프라이트 클래스 — `Player`, `Bullet`, `Enemy` |
| `settings.py` | 화면 크기·속도·색 같은 상수 |
| `assets/` | 그림 — ansimuz CC0 픽셀 아트 (출처: `assets/CREDITS.md`) |
| `AI_USAGE.md` | 생성형 AI 사용 기록 |

## 구현한 기능

- Pygame은 화면 왼쪽 위를 원점 `(0, 0)`으로 사용하여 오른쪽으로 이동할수록 X좌표가 증가하고 아래쪽으로 이동할수록 Y좌표가 증가하는 좌표계를 기준으로 객체의 위치를 처리합니다.

### [✓] F1 플레이어 이동

- `entities.py` 의 `Player.update(keys)` 에서 왼쪽 방향키를 누르고 있으면 `self.rect` 의 x값을 `PLAYER_SPEED` 만큼 감소시켜 왼쪽으로, 오른쪽 방향키를 누르고 있으면 같은 값만큼 증가시켜 오른쪽으로 이동시킵니다.
- `self.rect.left`가 0 이상이고 `self.rect.right`가 `WIDTH` 이하가 되도록 위치를 제한하여 플레이어가 화면 밖으로 이동하지 않도록 합니다.

### [✓] F2 총알 발사

- `entities.py` 의 `Player.shoot()` 에서 플레이어의 가로 가운데, 위쪽 경계를 기준으로 총알 객체를 생성하여 반환합니다.
- `Bullet.update()`에서 `self.rect.y`를 `BULLET_SPEED`만큼 감소시켜 총알을 위쪽으로 이동시킵니다. `self.rect.bottom`이 0보다 작아져 총알이 화면 위쪽을 완전히 벗어나면 스프라이트 그룹에서 제거하여 총알을 소멸시킵니다.
- `main.py`의 `Game.handle_events()`에서 게임 중 Space 키의 `KEYDOWN` 이벤트가 발생하면 `self.player.shoot()`를 호출합니다. 반환된 총알 객체를 `self.bullets`와 `self.all_sprites`에 추가하여 각각 충돌 판정 및 화면 출력에 활용합니다.

### [✓] F3 적 명중과 점수

- `main.py`의 `Game.update()`에서 `pygame.sprite.groupcollide()`를 사용하여 총알과 적의 충돌을 판정하고, 충돌한 총알과 적을 스프라이트 그룹에서 제거합니다. 이 함수는 충돌한 총알 객체를 키(key), 해당 총알과 충돌한 적 객체들의 리스트를 값(value)으로 갖는 딕셔너리를 반환합니다. 여러 총알이 같은 적과 동시에 충돌하면 한 적이 중복으로 기록될 수 있으므로, 적 객체를 집합(`set`)에 모아 중복을 제거합니다. 이후 충돌한 서로 다른 적의 수에 `SCORE_PER_ENEMY`를 곱한 만큼 점수를 증가시킵니다.
- `Game.draw()` 에서 현재 점수를 화면 왼쪽 위에 표시합니다.

### [✓] F4 피격과 게임 오버

- `main.py` 의 `Game.update()` 에서 `pygame.sprite.spritecollide()` 를 사용하여 플레이어와 적의 충돌을 기록합니다. 반환된 리스트에 포함된 적의 수만큼 `self.lives`를 감소시킵니다.
- `enemy.rect.top` 값이 화면 크기보다 커지면 스프라이트 그룹에서 제거하여 소멸시키고 `Game.lives` 인스턴스 속성 값을 감소시킵니다.
- `self.lives`값이 0 이하가 되면 `Game.end_game()`을 호출하여 게임 오버 상태로 전환합니다.
- `Game.draw_game_over()` 에서 화면 가운데에 **GAME OVER** 텍스트와 이번 게임의 점수를 표시합니다.

### [✓] F5 재시작과 최고 점수

- `load_high_score()` 에서 `HIGHSCORE_FILE` 에 지정된 파일을 읽고 저장된 최고 점수를 반환합니다. 파일이 없거나 저장된 값을 정수로 변환할 수 없는 경우에는 0을 반환합니다.
- `save_high_score()` 에서 전달받은 점수를 문자열로 변환하여 최고 점수 파일에 저장합니다. 파일이 없으면 새로 생성하고 이미 존재하면 기존 내용을 덮어씁니다.
- `Game.handle_events()` 에서 게임 오버 상태일 때 R 키를 누르면 Game.reset()을 호출하여 게임을 재시작합니다. 이때 플레이어, 적, 총알, 현재 점수와 생명을 초기화하며 최고 점수는 유지합니다.
- `Game.draw_game_over()` 에서 최고 점수와 0.5초마다 깜빡이는 **Press R to restart** 텍스트를 표시합니다.