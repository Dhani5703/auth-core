# Auth Core

로그인 모듈의 공통 라이브러리입니다. **플랫폼 독립적인 인증 모듈**입니다. 웹, 모바일, 백엔드 등 모든 환경에서 동일한 인증 로직을 사용할 수 있도록 설계되었습니다.

## 🚀 주요 특징

- **🔧 플랫폼 독립성**: HTTP 클라이언트와 토큰 저장소를 주입받아 사용
- **⚙️ 설정 가능한 API**: 환경별 API 엔드포인트 설정 지원
- **🛡️ 타입 안전성**: TypeScript로 완전한 타입 지원
- **📦 모듈화**: 필요한 기능만 선택적으로 사용 가능

## 📦 설치

```bash
npm install auth-core
```

## 🏗️ 프로젝트 구조

```
auth-core/
├── api/                          # API 호출 로직
│   ├── interfaces/               # HTTP 클라이언트 인터페이스
│   │   └── HttpClient.ts        # 플랫폼 독립적 HTTP 클라이언트
│   ├── utils/                   # 공통 유틸리티
│   │   ├── httpUtils.ts         # HTTP 요청 처리 로직
│   │   └── configUtils.ts       # API 설정 유틸리티
│   ├── emailAuthApi.ts          # 이메일 인증 API
│   ├── googleAuthApi.ts         # 구글 인증 API
│   └── index.ts                 # API 모듈 진입점
│
├── providers/                    # 로그인 방식 전략 모음 - 인증 제공자 (Strategy Pattern)
│   ├── interfaces/               # 인터페이스 정의
│   │   └── AuthProvider.ts      # 공통 인터페이스
│   ├── base/                    # 기본 클래스
│   │   └── BaseAuthProvider.ts  # 공통 로직 추상 클래스
│   ├── implementations/          # 구체적 구현체
│   │   ├── EmailAuthProvider.ts # 이메일 로그인
│   │   └── GoogleAuthProvider.ts# 구글 로그인
│   └── index.ts                 # Provider 모듈 진입점
│
├── factories/                    # 전략 객체 생성 책임 (Factory)
│   ├── AuthProviderFactory.ts   # 인증 제공자 생성
│   ├── TokenStoreFactory.ts     # 토큰 저장소 생성
│   └── AuthManagerFactory.ts    # AuthManager 생성
│
├── storage/                      # 토큰 저장소 (인터페이스만)
│   ├── TokenStore.interface.ts  # 저장소 인터페이스
│   ├── FakeTokenStore.ts        # 테스트용 가짜 저장소
│   └── index.ts                 # Storage 모듈 진입점
│
├── AuthManager.ts                # 인증 플로우 제어 (전략들을 주입 받아 로그인 흐름 제어)
├── types.ts                      # 공통 타입 정의
└── index.ts                      # 진입점 export
```

## 🎯 빠른 시작

### 기본 사용법

```typescript
import { 
  createAuthManager, 
  AuthManagerConfig 
} from 'auth-core';

// API 설정 (서비스에서 정의)
const apiConfig = {
  apiBaseUrl: 'https://api.myservice.com',
  endpoints: {
    login: '/api/v1/login',
    logout: '/api/v1/logout',
    refresh: '/token/refresh',
    me: '/user/me',
    requestVerification: '/email/verify',
    validate: '/token/validate',
    health: '/health'
  },
  timeout: 10000,
  retryCount: 3
};

// AuthManager 설정
const authManagerConfig: AuthManagerConfig = {
  providerType: 'email',
  apiConfig
};

// AuthManager 생성
const authManager = createAuthManager(authManagerConfig);

// 로그인 사용
const result = await authManager.login({
  provider: 'email',
  email: 'test@example.com',
  verificationCode: '123456'
});
```

### 커스텀 엔드포인트 설정

```typescript
// 백엔드 개발자가 다른 엔드포인트를 사용할 때
const customApiConfig = {
  apiBaseUrl: 'https://api.myservice.com',
  endpoints: {
    login: '/user/signin',           // 백엔드 마음대로
    logout: '/user/signout',         // 백엔드 마음대로
    requestVerification: '/email/send-code', // 백엔드 마음대로
    refresh: '/token/refresh',
    validate: '/token/validate',
    me: '/user/profile',
    health: '/status'
  },
  timeout: 15000,
  retryCount: 5
};
```

## 📚 주요 기능

### 인증 플로우
- 이메일 인증 요청
- 로그인/로그아웃
- 토큰 검증 및 갱신
- 사용자 정보 조회

### 토큰 관리
- 토큰 저장/조회/삭제
- 토큰 만료 확인
- 자동 토큰 갱신

### 플랫폼 지원
- 웹 프론트엔드 (localStorage)
- 모바일 앱 (SecureStore)
- 웹 백엔드 (세션/쿠키)

## 🔌 플랫폼별 구현

각 플랫폼에서는 다음을 구현해야 합니다:

### HTTP 클라이언트
```typescript
interface HttpClient {
  request(config: HttpRequestConfig): Promise<HttpResponse>;
}
```

### 토큰 저장소
```typescript
interface TokenStore {
  saveToken(token: Token): Promise<boolean>;
  getToken(): Promise<Token | null>;
  removeToken(): Promise<boolean>;
  // ... 기타 메서드들
}
```

## 📖 문서

- **[사용 가이드](docs/USAGE_GUIDE.md)**: 상세한 사용법과 예시
- **[아키텍처 문서](docs/ARCHITECTURE.md)**: 설계 원칙과 코드 흐름
