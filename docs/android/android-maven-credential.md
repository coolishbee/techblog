---
comments: true
---

# maven 자격증명

갑자기 잘 쓰고 있던 배포 명령어가 먹히지 않고 이러한 메시지를 응답받고 있었다.

```
Execution failed for task ':lib:module:publishReleasePublicationToMavenRepository'.
> Failed to publish publication 'release' to repository 'maven'
   > Could not PUT 'https://s01.oss.sonatype.org/content/repositories/snapshots/io/sdk/core/2.28.0-SNAPSHOT/maven-metadata.xml'. Received status code 401 from server: Content access is protected by token
```

메시지 내용만 봤을땐 자격증명에 문제가 있다는 건데 별도에 변경사항이 없으므로 이해가 가질 않았다.
gpg 키도 변경해보고 여러가지 시도를 해보았으나 당최 해결이 안됐다.

그러나 구글검색중에 maven 인증절차가 토큰방식으로 바뀐다는 글을 우연히 본 것 같은데
그게 설마 ossrh의 password 일줄은 몰랐다.

```
ossrhUsername=KeyID
ossrhPassword=Token
```

nexus repo manager 에서 Profile - User Token 에 xml 형태로 keyID 와 Token 이 있다.