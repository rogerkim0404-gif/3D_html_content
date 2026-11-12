# Three.js 3D 디지털 트윈 프로젝트 분석

본 문서는 `smart_classroom_camera_stable_2.html`(스마트 교실)과 `ThreeJS_korean_bus.html`(전기 시내버스 조립 공장) 두 개의 단일 HTML 파일로 구성된 Three.js 웹 애플리케이션의 실행 방법과 코드 구조를 분석한 문서입니다.

---

## 1. 프로젝트 실행 방법

두 파일 모두 외부 3D 모델(GLTF/OBJ 등)이나 이미지 텍스처 에셋을 다운로드하지 않고, 코드로 직접 도형을 생성(Procedural Generation)하도록 설계되어 있습니다. 또한 Three.js 라이브러리를 CDN을 통해 가져옵니다(`importmap` 사용).

### 실행 방법 A: 로컬 웹 서버 사용 (권장)
ES Module(`type="module"`)을 사용하고 있기 때문에, 브라우저의 보안 정책(CORS)으로 인해 단순히 파일을 더블 클릭해서 열면(file:// 프로토콜) 스크립트가 차단될 수 있습니다. 반드시 로컬 웹 서버를 구동하여 실행하는 것을 권장합니다.

1. **VS Code 확장 프로그램 사용 (가장 쉬움)**
   - Visual Studio Code 편집기에서 `Live Server` 확장 프로그램을 설치합니다.
   - HTML 파일을 열고 우측 하단의 `Go Live` 버튼을 클릭하거나, 에디터 화면을 우클릭하여 `Open with Live Server`를 선택합니다.

2. **Python 내장 서버 사용 (파이썬이 설치된 경우)**
   - 터미널(또는 명령 프롬프트)을 열고 해당 파일이 있는 폴더로 이동합니다.
   - 다음 명령어를 입력하여 서버를 실행합니다: `python -m http.server 8000`
   - 브라우저를 열고 `http://localhost:8000`에 접속하여 HTML 파일을 클릭합니다.

3. **Node.js 사용**
   - 터미널에서 `npx http-server`를 입력하여 서버를 구동한 후 브라우저로 접속합니다.

### 실행 방법 B: 웹 호스팅
- GitHub Pages, Vercel, Netlify 등의 정적 웹 호스팅 서비스에 파일을 업로드하여 바로 배포 및 실행할 수 있습니다.

---

## 2. 전체적인 문법 및 코드 구조

두 파일 모두 **HTML + CSS + JavaScript(Three.js)** 가 하나의 파일에 모두 포함된 SPA(Single Page Application) 형태를 취하고 있으며, 구조는 다음과 같이 크게 3개 영역으로 나뉩니다.

### 2.1. HTML 구조 (UI 레이아웃)
`<head>`와 `<body>`로 구성되며, 3D 캔버스가 렌더링될 영역과 사용자가 조작할 UI 패널이 분리되어 있습니다.

*   **`<script type="importmap">`**: 브라우저가 ES 모듈을 해석할 때 `three`라는 이름의 모듈을 어떤 CDN 주소(예: jsdelivr)에서 가져올지 매핑합니다.
*   **UI 컨테이너 (`<aside>`, `<div>` 등)**: 좌측/우측 패널, 하단 진행바, 상단 헤더 등 UI 요소들이 겹겹이 배치되어 있습니다. (스마트 교실은 환경 제어 슬라이더, 버스는 조립 공정 버튼 등)
*   **`#world` 또는 `#stage`**: 실제 Three.js의 `<canvas>` 엘리먼트가 주입되는 전체 화면 크기의 빈 컨테이너입니다.
*   **대체 화면 (Fallback/Loading)**: 3D 모델을 불러오는 동안 보여줄 로딩 화면과, WebGL 로드 실패 시 보여줄 2D 안내/에러 화면이 마크다운 내부에 하드코딩되어 있습니다.

### 2.2. CSS 구조 (스타일링)
`<style>` 태그 내부에 작성되어 있으며, 최신 CSS 기능들을 활용하여 모던한 UI를 구성합니다.

*   **CSS 변수 (`:root`)**: 테마 색상(배경, 패널, 텍스트, 포인트 컬러)을 변수로 지정하여 일관성 있게 관리합니다.
*   **레이아웃 (Grid/Flexbox)**: `#app` 전체 레이아웃을 `display: grid`로 나누거나, 각종 버튼 배열을 `flex`로 정렬합니다.
*   **유리 질감 (Glassmorphism)**: `backdrop-filter: blur(...)` 속성을 사용하여 3D 화면 위에 떠 있는 UI 패널을 반투명하고 고급스럽게 표현합니다.
*   **반응형 디자인 (`@media`)**: 화면 크기에 따라 패널 크기를 줄이거나, 모바일 화면에서는 하단/우측 패널을 숨기고 햄버거 메뉴를 통해 접근하도록 처리되어 있습니다.

### 2.3. JavaScript 구조 (Three.js 및 로직)
`<script type="module">` 내부에 작성되어 있으며, 전체적인 흐름은 다음과 같습니다.

1. **모듈 임포트**: 
   ```javascript
   import * as THREE from 'three';
   import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
   ```
2. **초기 셋업 (Scene, Camera, Renderer)**:
   - `THREE.Scene()`: 3D 공간 생성 및 배경/안개(Fog) 설정.
   - `THREE.PerspectiveCamera()`: 카메라 시야각, 위치 설정.
   - `THREE.WebGLRenderer()`: 렌더러 생성, 그림자(Shadow Map) 활성화, 톤 매핑(Tone Mapping) 및 해상도 최적화 설정. (Context Loss 등 그래픽 끊김 방지 로직 포함)
   - `OrbitControls`: 마우스/터치로 카메라를 회전, 확대, 패닝할 수 있도록 설정.
3. **조명 (Lighting)**:
   - `HemisphereLight`(주변광), `DirectionalLight`(태양광/그림자 생성), `PointLight` 등을 배치하여 실사풍의 환경을 조성합니다.
4. **절차적 모델링 (Procedural Modeling)**:
   - 외부 3D 파일을 로드하지 않고, 코드 내에서 `BoxGeometry`, `CylinderGeometry`, `SphereGeometry` 등을 조합하여 가구, 버스 부품, 건축물을 직접 그립니다.
   - 공통으로 쓰이는 재질(Material)은 메모리 절약을 위해 미리 변수로 선언하고 공유합니다.
   - 동적 텍스처(CanvasTexture)를 사용하여 전광판이나 온도 라벨 등에 텍스트를 실시간으로 그립니다.
5. **상태 관리 및 UI 연동**:
   - `state` 객체나 전역 변수를 두어 현재 조립 단계, 실내 온도, 재실 인원 등의 상태를 관리합니다.
   - `document.getElementById().addEventListener`를 통해 UI 버튼이나 슬라이더 값을 읽어와 3D 모델의 가시성(`visible`), 색상, 위치를 업데이트합니다.
6. **애니메이션 루프 (Animation Loop)**:
   - `renderer.setAnimationLoop` (또는 `requestAnimationFrame`) 내부에서 매 프레임마다 호출되는 로직입니다.
   - 카메라 이동 부드럽게 처리(Lerp), 공기 흐름/파티클 애니메이션, 버스 부품의 분해/조립 이동(Tweening) 등을 프레임마다 계산하여 화면을 갱신(`renderer.render`)합니다.