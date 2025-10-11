# 폼 인증 필터 UsernamePasswordAuthenticationFilter

- 스프링 시큐리티는 AbstractAuthenticationProcessingFilter 클래스를 사용자의 자격 증명을 인증하는 기본 필터로 사용한다.
- UsernamePasswordAuthenticationFilter는 AbstractAuthenticationProcessingFilter의 확장 클래스로 HttpServletRequest에서 제출된 사용자 이름과 암호를 추출하여 인증을 수행한다.
- 인증 프로세스가 초기활될 때 로그인 페이지와 로그아웃 페이지 생성을 위 DefaultLoginPageGeneratingFilter와 DefaultLogoutPageGeneratingFilter가 초기화된다.
