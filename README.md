# Kotlin LSP Plugin for Claude Code

JetBrains 공식 [Kotlin LSP](https://github.com/Kotlin/kotlin-lsp) 서버를 Claude Code와 연결하는 플러그인입니다.

## 사전 요구사항

### 1. Java 17 이상
Kotlin LSP는 Java 17 이상이 필요합니다.

```bash
java -version
```

### 2. Kotlin LSP 설치

**Homebrew (권장):**
```bash
brew install JetBrains/utils/kotlin-lsp
```

**수동 설치:**
1. [Kotlin LSP Releases](https://github.com/Kotlin/kotlin-lsp/releases)에서 standalone zip 다운로드
2. 압축 해제 후 실행 권한 부여:
   ```bash
   chmod +x kotlin-lsp.sh
   ```
3. PATH에 추가하거나 심볼릭 링크 생성:
   ```bash
   ln -s /path/to/kotlin-lsp.sh /usr/local/bin/kotlin-lsp
   ```

## 플러그인 설치

### Marketplace를 통한 설치 (권장)

1. **마켓플레이스 추가:**
   ```
   /plugin marketplace add h16rkim/cc-kotlin-lsp-plugin
   ```

2. **플러그인 설치:**
   ```
   /plugin install kotlin-lsp@kotlin-lsp-marketplace
   ```

### 로컬 설치

**테스트용:**
```bash
claude --plugin-dir /path/to/cc-kotlin-lsp-plugin
```

**영구 설치:**
```bash
# 플러그인 디렉토리에 심볼릭 링크 생성
ln -s /path/to/cc-kotlin-lsp-plugin ~/.claude/plugins/kotlin-lsp
```

## 지원 기능

Kotlin LSP가 제공하는 기능:

- ✅ Gradle JVM 프로젝트 임포트
- ✅ Semantic 하이라이팅
- ✅ Go to Definition (Kotlin/Java 간 네비게이션)
- ✅ Find References
- ✅ Code actions (quickfixes, organize imports)
- ✅ Rename 리팩토링
- ✅ 실시간 진단 및 자동완성
- ✅ Document symbols (Outline)
- ✅ Hover 정보

## 지원 파일 확장자

- `.kt` - Kotlin 소스 파일
- `.kts` - Kotlin 스크립트 파일

## 플랫폼 지원

- ✅ macOS (공식 지원)
- ✅ Linux (공식 지원)
- ⚠️ Windows (부분 지원)

## 알려진 제한사항

Kotlin LSP는 현재 실험적 단계입니다:

- Gradle KMP(Kotlin Multiplatform) 프로젝트 미지원 (개발 중)
- Code formatting 미지원
- Windows 완전 지원 미비

## 문제 해결

### LSP 서버가 시작되지 않는 경우

1. kotlin-lsp가 PATH에 있는지 확인:
   ```bash
   which kotlin-lsp
   ```

2. Java 버전 확인:
   ```bash
   java -version  # 17 이상이어야 함
   ```

3. 로그 확인:
   ```bash
   claude --enable-lsp-logging --plugin-dir /path/to/cc-kotlin-lsp-plugin
   ```
   로그 파일 위치: `~/.claude/debug/`

### Gradle 프로젝트가 인식되지 않는 경우

- 프로젝트 루트에 `build.gradle` 또는 `build.gradle.kts` 파일이 있는지 확인
- Gradle wrapper (`gradlew`)가 있는지 확인

### 플러그인 설치 문제

플러그인이 인식되지 않는 경우:
```bash
# 플러그인 유효성 검사
/plugin validate /path/to/cc-kotlin-lsp-plugin
```

## 참고 자료

- [Kotlin LSP GitHub](https://github.com/Kotlin/kotlin-lsp)
- [Claude Code Plugins Documentation](https://code.claude.com/docs/en/plugins)
- [LSP Server Reference](https://code.claude.com/docs/en/plugins-reference#lsp-server)
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)

## 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.
