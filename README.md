# 데이터 저널리즘 실습 기록

[수업 원본 코드](https://github.com/journal-izm/ai-datajournalism-labs)를 바탕으로 실행한 학습용 저장소입니다. 현재 범위는 **환경 확인 → 시각화 기초 → 빅데이터 차트 문법(2-3 포함)**입니다. 이후 실습은 진행한 뒤 추가합니다.

그래프·표가 저장된 출력, 캡처, 결과 폴더와 가상환경은 제외했습니다. 아래 환경과 입력 데이터를 준비한 뒤 노트북을 실행하면 결과가 생성됩니다. 데이터·패키지·실행 장치가 다르면 수치나 소요 시간이 달라질 수 있습니다.

## 1. 준비

- Windows 기준 안내입니다. Python **3.12**, VS Code, Microsoft의 **Python**, **Jupyter** 확장을 설치합니다.
- GitHub의 **Code → Download ZIP**으로 받은 뒤 압축을 풀고, VS Code에서 `README.md`와 `requirements.txt`가 있는 폴더를 엽니다.
- **Terminal → New Terminal**에서 PowerShell을 엽니다. 다음 명령은 모두 프로젝트 최상위 폴더에서 실행합니다.

## 2. 제외된 가상환경을 내 컴퓨터에서 다시 만들기

가상환경은 내려받아 실행하는 파일이 아니라 각 컴퓨터에서 새로 만드는 전용 Python 환경입니다. 이름만 `.venv`여도 Python 버전은 다를 수 있습니다. 여기서는 다른 환경과 구별하려고 `.venv312`를 사용합니다.

아래 명령을 **한 줄씩**, 앞 명령이 끝난 뒤 실행합니다.

```powershell
py -3.12 --version
py -3.12 -m venv .venv312
.\.venv312\Scripts\python.exe -m pip install --upgrade pip
```

`py -3.12`가 없으면 [Python 공식 사이트](https://www.python.org/downloads/)에서 3.12를 설치합니다. 이미 만든 가상환경의 Python 버전은 이름을 바꿔도 바뀌지 않습니다.

### PyTorch 설치: CPU 또는 NVIDIA GPU 중 하나 선택

2-3은 PyTorch가 필요합니다. GPU가 없어도 CPU로 실행할 수 있지만 더 오래 걸리고 GPU 속도 비교는 할 수 없습니다.

**CPU로 실행할 경우:**

```powershell
.\.venv312\Scripts\python.exe -m pip install torch --index-url https://download.pytorch.org/whl/cpu
```

**NVIDIA GPU로 실행할 경우:** [PyTorch 공식 설치 페이지](https://pytorch.org/get-started/locally/)에서 Windows / Pip / Python과 GPU·드라이버에 맞는 CUDA 옵션을 선택합니다. 안내된 명령의 `pip3` 대신 `.\.venv312\Scripts\python.exe -m pip`를 사용합니다. 수업 원본 README의 RTX 50 시리즈용 CUDA 12.8 예시는 다음과 같습니다.

```powershell
.\.venv312\Scripts\python.exe -m pip install torch --index-url https://download.pytorch.org/whl/cu128
```

그다음 공통 패키지를 설치합니다. GPU 사용 시에는 GPU용 PyTorch를 먼저 설치해 의도하지 않은 빌드가 설치되는 일을 피합니다.

```powershell
.\.venv312\Scripts\python.exe -m pip install -r requirements.txt
.\.venv312\Scripts\python.exe -m pip check
.\.venv312\Scripts\python.exe -c "import sys, torch; print(sys.executable); print(torch.__version__); print('CUDA build:', torch.version.cuda); print('GPU available:', torch.cuda.is_available())"
```

GPU 실행을 선택했다면 `GPU available: True`인지 확인합니다. `nvidia-smi`에서 GPU 이름이 보이는 것만으로 PyTorch의 GPU 사용까지 확인된 것은 아닙니다. False이면 NVIDIA 드라이버, 설치한 PyTorch 빌드와 선택한 환경을 확인합니다. 전체 학습이 필요 없는 이 범위에는 Whisper·ffmpeg·Ollama·n8n이 필요하지 않습니다.

이 안내는 환경 재구성용이며 모든 컴퓨터에서 동일한 버전을 보장하는 고정 버전 목록은 아닙니다.

## 3. 노트북에서 가상환경 선택

터미널의 `(.venv)` 표시와 노트북이 실행하는 환경은 별개입니다. **각 노트북에서** 다음을 확인합니다.

1. `.ipynb` 파일을 열고 오른쪽 위 환경 이름을 클릭합니다.
2. **Select Another Kernel → Python Environments → `.venv312` (Python 3.12)**를 선택합니다.
3. 목록에 없으면 **Enter interpreter path / Find**에서 프로젝트 안의 `.venv312\Scripts\python.exe`를 선택합니다.
4. `ipykernel`이 없다는 안내가 나오면 설치합니다. 직접 설치하려면 아래 명령을 사용합니다.

```powershell
.\.venv312\Scripts\python.exe -m pip install ipykernel
```

5. 환경을 바꾸거나 패키지를 설치했다면 **Restart → Run All**로 처음부터 실행하고 `Ctrl + S`로 저장합니다.

실행 환경은 다음 셀로 확인할 수 있습니다.

```python
import sys
print(sys.version)
print(sys.executable)  # 프로젝트의 .venv312 안을 가리키는지 확인
```

[VS Code 공식 커널 선택 안내](https://code.visualstudio.com/docs/datascience/jupyter-kernel-management)

## 4. 입력 데이터 준비

`2_빅데이터_차트문법` 폴더에 입력 파일을 둡니다. 입력 데이터는 실행으로 생성되는 결과 파일과 다릅니다.

### 미세먼지 CSV: 함께 제공

`data_air_2022.csv`는 수업에서 받은 입력 데이터이며 **CP949** 인코딩입니다. 열은 `location, day, so2, co, o3, no2, pm10, pm25`입니다. 종로구 자료 673행이며 실제 날짜는 **2022-04-01~2024-02-15**로, 파일 이름보다 넓습니다.

수업 ZIP에서 같은 파일을 다시 받을 수도 있습니다. 원본 코드 저장소에는 확인 당시 CSV가 없어, 이 저장소에는 실행에 필요한 수업 CSV를 포함했습니다. 데이터의 최초 수집 기관과 가공 경로는 제공 파일만으로 확인되지 않았습니다.

**2022년 기사와 비교할 때는 두 데이터의 기간을 맞추세요.** 아래 코드를 2-1의 `air['day']` 날짜 변환 뒤, 병합 전에 넣으면 수업 기사 기간으로 제한할 수 있습니다. 포함된 노트북은 인코딩 수정 외에는 수업 분석 흐름을 유지했습니다.

```python
air = air[air['day'].between('2022-04-01', '2022-12-31')].copy()
```

### 기사 엑셀: 직접 준비

수업 파일 `NewsResult_20220401-20221231.xlsx`는 기사 20,000행이며 기사 본문·URL 등을 포함합니다. 이 저장소에는 엑셀을 재업로드하지 않았습니다. **수업 ZIP의 원본 파일을 사용하면 수업 결과와 비교하기 쉽습니다.**

다른 자료로 실행하려면 [빅카인즈](https://www.bigkinds.or.kr/)의 **뉴스검색·분석**에서 다음과 같이 준비합니다.

1. 필요한 경우 로그인합니다.
2. 검색어 `미세먼지`, 기간 **2022-04-01~2022-12-31**로 검색합니다.
3. 검색 결과를 엑셀로 내려받습니다. 사이트의 다운로드 제한·절차에 따라 기간을 나눠 받을 수 있습니다.
4. `일자`(예: `20220405`), `제목` 열이 있는지 확인합니다.
5. 파일 이름을 `NewsResult_*.xlsx` 형태로 두고 `2_빅데이터_차트문법`에 넣습니다. 여러 파일은 노트북에서 합쳐 읽습니다. 같은 파일의 사본을 중복해서 넣지 마세요.

수업 엑셀의 상세 검색 조건은 확인되지 않았으므로 위 방법은 대체 자료 준비 방법입니다. 언론사·검색 조건·데이터 갱신·다운로드 범위에 따라 수업의 기사 수와 그래프는 달라질 수 있습니다.

1번 시각화와 2-2는 Seaborn 예제 데이터를 인터넷에서 받습니다. 2-3은 Hugging Face에서 한국어 문장 모델을 처음 내려받으므로 인터넷 연결과 대기 시간이 필요합니다.

## 5. 실행 순서

| 순서 | 파일 | 확인할 것 |
|---|---|---|
| 0 | `0_환경확인/check_env.ipynb` | Python 경로·패키지·글꼴 |
| 1 | `1_시각화기초/visualization_basics.ipynb` | 기본 그래프와 한글 |
| 2-1 | `2_빅데이터_차트문법/1_bigdata_news_vs_air.ipynb` | 기사·미세먼지 표와 그래프 |
| 2-2 | `2_빅데이터_차트문법/2_chart_grammar.ipynb` | 축·증감률·강조·제목 비교 |
| 2-3 | `2_빅데이터_차트문법/3_gpu_title_clusters.ipynb` | 문장 임베딩·기사 묶음·막대그래프 |

셀은 위에서부터 실행합니다. 2-3에서 대표 제목을 읽고 `names`의 `묶음0` 등을 실제 주제 이름으로 바꾸는 부분은 직접 수행하는 실습입니다. 결과를 저장하려면 실행 후 `Ctrl + S`를 누릅니다.

## 6. 실습 중 겪은 오류와 해결

| 증상 | 확인·해결 |
|---|---|
| `UnicodeDecodeError: 'utf-8' codec can't decode ...` | 제공 CSV는 CP949입니다. `pd.read_csv('data_air_2022.csv', encoding='cp949')`로 읽습니다. 업로드용 2-1에는 반영했습니다. 다른 CSV는 실제 인코딩에 맞춥니다. |
| `No module named 'matplotlib'`, `'torch'` 등 | 새 가상환경은 패키지가 별도로 필요합니다. 위 설치 명령으로 **같은 환경**에 설치한 뒤 커널을 다시 시작합니다. |
| 설치했는데도 모듈이 없다고 나옴 | 터미널과 노트북의 Python이 다를 수 있습니다. `sys.executable`을 확인하고 `.venv312`를 다시 선택합니다. |
| Colab 설치 셀이 초록 체크인데 torch가 없음 | `if IN_COLAB:` 부분은 VS Code에서 설치하지 않습니다. PowerShell에서 PyTorch를 설치합니다. |
| `.venv`인데 Python 3.14로 표시됨 | 폴더 이름은 버전을 보장하지 않습니다. 기존 환경은 유지하고 `py -3.12 -m venv .venv312`로 만든 환경을 선택합니다. |
| `Workbook contains no default style` | openpyxl의 엑셀 서식 경고입니다. 이후 데이터가 읽히면 실행 실패가 아닙니다. |
| `No objects to concatenate` | 기사 엑셀이 검색되지 않았을 수 있습니다. 폴더 위치와 `NewsResult_*.xlsx` 파일명을 확인합니다. |
| GPU 이름은 나오지만 `사용 장치: cpu` | PyTorch의 CUDA 사용 여부는 별도입니다. `torch.version.cuda`, `torch.cuda.is_available()`와 GPU 드라이버를 확인합니다. |
| 설치 후에도 이전 빨간 오류가 남음 | 출력은 자동 갱신되지 않습니다. 설치 완료 후 Restart → 첫 셀부터 다시 실행합니다. |
| pip 업데이트 안내·커널 재시작 안내 | 안내 문구 자체는 설치 실패가 아닙니다. 오류 여부와 설치 종료를 확인하고 커널을 다시 시작합니다. |

`(.venv312)` 활성화는 선택 사항입니다. 위 명령들은 Python 경로를 직접 지정하므로 활성화 스크립트 실행 정책을 바꿀 필요가 없습니다. 선택적으로 활성화하려면 PowerShell에서 `.\.venv312\Scripts\Activate.ps1`, 종료는 `deactivate`입니다. 노트북 커널 선택은 별도로 해야 합니다.

## 7. 제출 캡처와 GitHub 업데이트

- 제출용 캡처에는 **파일 이름·실행 결과·오른쪽 위 가상환경**이 함께 보이게 합니다. 터미널은 `Ctrl + J`로 접어도 됩니다.
- 긴 노트북은 읽을 수 있는 크기로 여러 장 찍습니다. 전체 노트북을 한 장으로 축소할 필요는 없습니다.
- GitHub에는 실행 코드를 올립니다. 업로드용 사본에서 **Clear All Outputs**를 실행하고 저장하세요. `.gitignore`는 노트북 안의 출력을 지우지 않습니다.
- `.venv*`, 생성 이미지·표·결과 폴더, 엑셀, 키 파일은 제외합니다. 필요한 입력 CSV만 예외로 포함했습니다.
- 다음 실습은 같은 저장소에 추가합니다. 로컬의 결과가 담긴 원본은 보관하고 업로드용 사본만 정리하세요.

## 출처

- 수업 코드: [journal-izm/ai-datajournalism-labs](https://github.com/journal-izm/ai-datajournalism-labs)
- 기사 데이터 준비: [빅카인즈](https://www.bigkinds.or.kr/)
- 설치·환경 안내: [PyTorch](https://pytorch.org/get-started/locally/), [VS Code](https://code.visualstudio.com/docs/datascience/jupyter-kernel-management)

원본 수업 코드와 데이터의 권리는 각 원저작자·제공자에게 있습니다. 이 저장소는 실습 범위를 정리하고 실행 안내 및 CSV 인코딩 수정을 추가한 학습 기록입니다.
