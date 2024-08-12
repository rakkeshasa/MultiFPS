# MultiFPS
멀티플레이 FPS 게임

---
## Online Subsystem
다른 플레이어와 게임을 하기 위해서는 하나의 IP주소에 플레이어들이 접속할 필요가 있다.</BR>
LAN 연결을 할 시 상대방의 IP를 알아야하고, 같은 라우터를 써야한다는 단점이 있다.</BR>
다른 라우터에 있는 즉, 전 세계의 플레이어와 같이 게임을 하기 위해서는 접속을 담당할 데디케이트 서버를 두거나 리슨 서버를 담당할 호스트의 IP 주소로 접속을 하는 것이다.</BR>
아니면 Steam과 xbox에서 제공하는 매치 메이킹 기능을 사용하여 연결하는 방법이 있는데, 이번 프로젝트는 이 방법을 사용할 것이다.</br></br>

마침 Unreal Engine에서는 이 매치 메이킹 기능을 사용할 수 있게 추상 레이어를 제공하는데 그것이 'Unreal Engine Subsystem'이다.</br>
장점은 각 플랫폼마다 같은 동작을 하는 코드를 고칠 필요 없이 재사용이 가능해 하나의 코드로 모든 플랫폼에서 제공하는 매치 메이킹 기능을 사용할 수 있다.</br>

Online Subsystem은 온라인 플랫폼 서비스(Steam, xbox)의 기능에 액세스하는 방법을 제공한다.</br>
각 플랫폼 서비스는 친구, 업적, 매치 메이킹 세션을 지원하는 자체 서비스 세트가 있으며, Online Subsystem은 각 플랫폼에 대해 이러한 서비스를 처리하도록 설계된 Interface가 포함되어 있다.</br>

여기서 가장 신경 쓸 부분은 다른 플레이어가 내가 플레이하는 곳에 참여하도록 하는 매치 메이킹 세션 인터페이스 즉, 세션 인터페이스(Session Interface)이다.</BR>
세션 인터페이스는 게임 세션을 생성, 관리, 파괴하는 것을 담당한다. 또한 세션 검색과 메치 메이킹 기능 검색도 다룬다.</br>
일반적인 게임 세션의 라이프 사이클은 세션 생성 - 플레이어 입장 대기 - 플레이어 등록 - 세션 시작 - 게임 플레이 - 세션 종료 - 플레이어 등록 해제 - 세션 파괴 또는 업데이트 순으로 이루어진다.</br>
이 중 세션에 대한 생성(Create), 찾기(Find), 참가(Join), 시작(Start), 파괴(Destroy)에 대해 자세히 살펴볼 것이다.</br></br>

만약 내가 호스트라면 세션 설정을 구성한 다음 세션 인터페이스 함수를 통해 CreateSession()을 호출한다.</br>
이후 플레이할 게임 레벨을 열고 다른 플레이어를 기다린다.</br></BR>

호스트가 아닌 플레이어는 검색 설정(참여할 세션만 구별해내는 설정)을 구성한 다음 인터페이스 함수 FindSessions()를 호출한다.</br>
함수가 유효한 세션을 추출하면 JoinSession()함수를 호출하고 해당 함수는 게임 참가할 IP 주소를 얻어내게 된다.</BR>
얻어낸 IP 주소를 통해 호스트가 열어둔 게임 레벨에 접속하여 플레이하기만 하면 된다.</BR></br>

그렇다면 스팀의 온라인 세션은 어떻게 가져올까?</br>
엔진에 Online Subsystem Steam을 설치하고 C++에서 해당 모듈로 작업하면 된다.</br>
C++로 해당 모듈을 액세스할려면 권한이 필요하므로 빌드 파일(cs파일)에 있는 PublicDependencyModuleNames에 OnlineSubsystemSteam과 OnlineSubsystem을 추가하면 된다.</br>
OnlineSubsystem은 OnlineSubsystemSteam과 상호 작용하는 전반적인 Subsystem으로 둘은 서로 살짝 다르다.</br>

이후 DefaultEngine.ini를 Online Subsystem을 사용할 수 있게 수정해준다</br>
수정법은 https://dev.epicgames.com/documentation/ko-kr/unreal-engine/online-subsystem-steam?application_version=4.27 해당 페이지에 상세히 나와있다.</br></br>

설정이 완료됐으면 기본플레이어 C++ 파일에서 OnlineSubsystem에 액세스해주면 된다.</br>

```
IOnlineSubsystem* OnlineSubsystem = IOnlineSubsystem::Get();
if (OnlineSubsystem)
{
	OnlineSessionInterface = OnlineSubsystem->GetSessionInterface();

	if (GEngine)
	{
		GEngine->AddOnScreenDebugMessage(
			-1,
			15.F,
			FColor::Blue,
			FString::Printf(TEXT("Found Subsystem %s"), *OnlineSubsystem->GetSubsystemName().ToString())
			);
	}
}
```
</br>
OnlineSubsystem 인터페이스에 액세스하고 인터페이스가 제공하는 함수를 이용해 지금 사용하는 Subsystem의 이름을 알아와 화면에 보여주고 있다.</br>
테스트를 하면서 이상한 점이 있었는데, 에디터에서 리슨 서버와 데디케이트 서버 방식으로 했을 때는 NULL이라고 떴는데, 패키징 이후 게임을 실행하니 Steam이라고 떴다.</br>
GetSessionInterface() 함수는 IOnlineSessionPtr 타입을 리턴하는데 받아주는 변수 OnlineSessionInterface는 IOnlineSessionPtr 타입으로 선언하면 안됐다.</br>
IOnlineSessionPtr이 typedef타입으로 선언되어 있어, <strong>TSharedPtr&lt;class IOnlineSession, ESPMode::ThreadSafe></strong> 타입으로 선언해줘야 했다.</br>
