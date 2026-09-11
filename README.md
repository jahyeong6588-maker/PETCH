# PETCH

반려동물과 함께 갈 수 있는 식당·카페를 찾는 검색 웹사이트입니다.
RUNCH(점심 추천 서비스)와 동일한 UI 스타일로 제작했습니다.

## 현재 상태
- 정적 HTML/CSS/JS 한 파일(`index.html`)로 동작합니다. 별도 빌드 과정이 없어서 Netlify에 그대로 배포하면 됩니다.
- **`index.html`은 배포용으로 완전한 문서 구조(`<!doctype>`+`<head>`+뷰포트 메타)를 갖춘 파일이에요.** 실제 모바일 브라우저에서 뷰포트 메타 태그가 없으면 화면이 축소되어 작게 보이는 문제가 있어서, 이 저장소의 `index.html`은 항상 전체 HTML 문서 형태로 유지합니다 (Claude 아티팩트 미리보기용 소스와는 별도로 관리돼요).
- 식당·카페 목록: 식품의약품안전처 식품안전나라(foodsafetykorea.go.kr)에 등록된 "반려동물 동반출입 음식점" 공공데이터 중 153곳을 사용합니다. 전국 17개 시/도를 모두 포함해요 (서울 46, 경기 4, 인천 8, 강원 4, 충북 8, 충남 8, 대전 5, 세종 3, 경북 8, 경남 8, 대구 8, 울산 5, 부산 4, 전북 7, 전남 14, 광주 5, 제주 3). 전화번호·편의시설(테라스/놀이공간 등) 정보는 원본 데이터에 없어서 비워뒀고, Supabase 기반 "정보 추가" 제보 기능으로 채워지도록 했습니다.
- "찾는 지역"은 시/도 → 시/군/구 → 읍/면/동 3단계 드롭다운이에요. 각 단계에서 "전체"를 선택하면 그 상위 범위 전체(예: 서울 전체, 서울 성동구 전체)로 검색할 수 있습니다.
- "📍 내 위치 주변" 모드도 있어요. 버튼을 누르면 브라우저 위치 권한을 요청하고, 허용하면 반경 20km 이내 업소를 가까운 거리순으로 보여줍니다 (실제 거리는 직선거리 기준 근사치예요). 확인된 내 위치는 위도/경도로 바로 표시되고, 카카오맵 SDK를 정상적으로 불러온 환경(예: Netlify 배포본)에서는 실제 주소로도 자동 업그레이드돼요.
- 지도: 카카오맵 JavaScript SDK를 연동해뒀습니다. SDK를 정상적으로 불러오면 실제 카카오 지도가 뜨고, 못 불러오면(예: claude.ai 프리뷰처럼 외부 스크립트가 막혀 있는 환경) 예시 지도 이미지로 자동 대체됩니다.
- "정보 추가": Supabase에 연결되어 있어서, 배포된 사이트에서는 실제로 방문자들이 남긴 제보(야외테라스/놀이공간/배변봉투 제공/주차장)가 쌓이고 카드에서 확인할 수 있습니다. **아래 SQL을 Supabase에서 한 번 실행해야 동작합니다.**

## 처음 배포 전에 해야 할 설정

### 1. 카카오 개발자 콘솔 — 도메인 등록
1. developers.kakao.com 로그인 → 내 애플리케이션 → 해당 앱 선택
2. 앱 설정 → 플랫폼 → Web 플랫폼 등록
3. Netlify에서 받은 주소(`https://xxxxx.netlify.app`)를 사이트 도메인으로 추가하고 저장

이 등록을 안 하면 지도 자리에 카카오 지도가 안 뜨고 예시 이미지만 계속 보입니다.

### 2. Supabase — 테이블 만들기
Supabase 프로젝트의 **SQL Editor**에서 아래 SQL을 한 번 실행하세요.

```sql
create table if not exists public.place_tags (
  id bigint generated always as identity primary key,
  place_name text not null,
  tag text not null,
  created_at timestamptz not null default now()
);

alter table public.place_tags enable row level security;

create policy "public can read tags"
  on public.place_tags for select
  using (true);

create policy "public can insert tags"
  on public.place_tags for insert
  with check (true);
```

지금은 로그인 기능이 없어서 누구나 읽고 쓸 수 있게 열어둔 상태예요. 나중에 사용자 로그인을 붙이면 스팸성 제보를 막기 위해 정책을 더 좁히는 걸 추천해요.

### 3. 키 관련 참고
- `index.html`에 들어있는 카카오 **JavaScript 키**와 Supabase **publishable 키**는 원래 브라우저(클라이언트)에 노출되는 걸 전제로 만들어진 "공개용" 키예요. 그대로 커밋해도 괜찮습니다.
- 반대로 카카오 **REST API 키**나 Supabase **service_role 키**처럼 "비밀" 키는 절대 이 파일이나 깃허브에 올리면 안 돼요. 그런 키가 필요한 기능은 별도 서버(또는 Supabase Edge Function) 쪽에서만 써야 합니다.

## 다음 단계 (예정)
- [x] 카카오맵 JavaScript SDK 연동
- [x] Supabase 연동 (사용자 제보 데이터 저장)
- [x] 실제 공공데이터(식품안전나라)로 업소 목록 교체 + 지역 3단계 드롭다운 + 내 위치 주변(20km) 검색
- [x] 전국 17개 시/도 데이터로 확장 (153곳) + 내 위치 좌표/주소 표시
- [x] 모바일 브라우저 뷰포트 버그 수정 (실제 폰 화면에 꽉 차게 보이도록)
- [ ] GitHub ↔ Netlify 자동 배포 연결
- [ ] 즐겨찾기를 기기 저장(localStorage)에서 계정 기반으로 전환 (로그인 기능 필요)
- [ ] 편의시설(테라스/놀이공간/주차장 등)·전화번호 정보 보강 (제보 누적 또는 추가 공공데이터 연계)

## 데이터 출처
- 업소 목록: [식품의약품안전처 식품안전나라 — 반려동물 동반 가능 업소](https://www.foodsafetykorea.go.kr/portal/petKorea.do)
