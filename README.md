# PETCH

반려동물과 함께 갈 수 있는 식당·카페를 찾는 검색 웹사이트입니다.
RUNCH(점심 추천 서비스)와 동일한 UI 스타일로 제작했습니다.

## 현재 상태
- 정적 HTML/CSS/JS 한 파일(`index.html`)로 동작합니다. 별도 빌드 과정이 없어서 Netlify에 그대로 배포하면 됩니다.
- 식당·카페 목록 자체는 아직 예시(mock) 데이터입니다.
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
- [ ] GitHub ↔ Netlify 자동 배포 연결
- [ ] 즐겨찾기를 기기 저장(localStorage)에서 계정 기반으로 전환 (로그인 기능 필요)
- [ ] 실제 카카오 장소 검색 API로 mock 데이터 교체
