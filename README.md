# Google ADK Python 퀵스타트 정리

Google **Agent Development Kit(ADK)** 를 Python으로 설치하고, 첫 에이전트를 만들어 실행하기까지의 과정을 정리한 노트입니다.
원문: [Python Quickstart for ADK](https://adk.dev/get-started/python/)

---

## 1. 준비물

| 항목 | 요구 사항 |
|------|-----------|
| Python | 3.10 이상 |
| 패키지 관리자 | `pip` |
| API 키 | Gemini API 키 ([Google AI Studio](https://aistudio.google.com/app/apikey)에서 발급) |

---

## 2. 가상환경 만들기 (권장)

```bash
python -m venv .venv
```

가상환경 활성화:

```bash
# Windows 명령 프롬프트(cmd)
.venv\Scripts\activate.bat

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

---

## 3. ADK 설치

```bash
pip install google-adk
```

> 이 저장소에서는 `pip install -r requirements.txt` 로도 설치할 수 있습니다.

---

## 4. 에이전트 프로젝트 생성

```bash
adk create my_agent
```

생성되는 폴더 구조:

```text
my_agent/
    agent.py      # 에이전트 메인 코드
    .env          # API 키 또는 프로젝트 ID
    __init__.py
```

- `agent.py` 안의 **`root_agent`** 가 ADK 에이전트에서 **유일하게 필수**인 요소입니다.
- 에이전트가 사용할 **도구(tool)** 는 일반 Python 함수로 정의해 `tools=[...]` 에 넘깁니다.

---

## 5. 에이전트 코드 작성 (`my_agent/agent.py`)

도시 이름을 받아 현재 시각을 돌려주는 `get_current_time` 도구(모의 구현)를 추가한 예제입니다.

```python
from google.adk.agents.llm_agent import Agent


# Mock tool implementation
def get_current_time(city: str) -> dict:
    """Returns the current time in a specified city."""
    return {"status": "success", "city": city, "time": "10:30 AM"}


root_agent = Agent(
    model="gemini-flash-latest",
    name="root_agent",
    description="Tells the current time in a specified city.",
    instruction=(
        "You are a helpful assistant that tells the current time in cities. "
        "Use the 'get_current_time' tool for this purpose."
    ),
    tools=[get_current_time],
)
```

### 코드 포인트

| 인자 | 의미 |
|------|------|
| `model` | 사용할 LLM (여기서는 Gemini Flash 최신 버전) |
| `name` | 에이전트 이름 |
| `description` | 에이전트가 하는 일에 대한 짧은 설명 (다른 에이전트가 위임할 때 참고) |
| `instruction` | 에이전트의 행동 지침(시스템 프롬프트) |
| `tools` | 에이전트가 호출할 수 있는 함수 목록 |

- 도구 함수의 **docstring과 타입 힌트** 가 LLM에게 도구 설명으로 전달되므로 명확하게 적는 것이 중요합니다.
- 반환값은 `dict` 로 주는 것이 일반적이며, `status` 같은 필드로 성공/실패를 알려주면 좋습니다.

---

## 6. API 키 설정

`my_agent/.env` 파일에 키를 넣습니다.

```bash
# macOS / Linux
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > my_agent/.env
```

```powershell
# Windows PowerShell
'GOOGLE_API_KEY="YOUR_API_KEY"' | Out-File -Encoding utf8 my_agent\.env
```

> 이 저장소에는 실제 키 대신 `my_agent/.env.example` 만 올려두었습니다. 복사해서 `.env` 로 이름을 바꾼 뒤 키를 넣으세요.
> `.env` 는 `.gitignore` 에 포함되어 있어 커밋되지 않습니다.

Gemini 외의 모델도 사용할 수 있습니다. → [Models & Authentication](https://adk.dev/agents/models)

---

## 7. 에이전트 실행

### 7-1. 명령줄(CLI)로 실행

```bash
adk run my_agent
```

### 7-2. 웹 UI로 실행

```bash
adk web --port 8000
```

- `my_agent/` 폴더가 **들어 있는 상위 폴더** 에서 실행해야 합니다.
  (예: `agents/my_agent/` 구조라면 `agents/` 에서 실행)
- 브라우저에서 `http://localhost:8000` 접속 → 왼쪽 위에서 에이전트 선택 → 질문 입력

> ⚠️ **ADK Web은 개발·디버깅 전용** 입니다. 실제 서비스(프로덕션) 배포용이 아닙니다.

---

## 8. 전체 흐름 요약

```text
가상환경 생성 → pip install google-adk → adk create my_agent
   → agent.py에 root_agent + 도구 작성 → .env에 GOOGLE_API_KEY 설정
   → adk run my_agent (CLI) 또는 adk web (웹 UI)
```

---

## 이 저장소 구조

```text
adk_python_quickstart/
├── README.md             # 이 정리 문서
├── requirements.txt      # google-adk
├── .gitignore
└── my_agent/
    ├── __init__.py
    ├── agent.py          # root_agent + get_current_time 도구
    └── .env.example      # API 키 예시 (실제 키는 .env에)
```
