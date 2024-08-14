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

![steamSession](https://github.com/user-attachments/assets/fa0af67e-fd81-48ee-bf1f-458c775ab9ba)

</br>
OnlineSubsystem 인터페이스에 액세스하고 인터페이스가 제공하는 함수를 이용해 지금 사용하는 Subsystem의 이름을 알아와 화면에 보여주고 있다.</br>
테스트를 하면서 이상한 점이 있었는데, 에디터에서 리슨 서버와 데디케이트 서버 방식으로 했을 때는 NULL이라고 떴는데, 패키징 이후 게임을 실행하니 Steam이라고 떴다.</br>
GetSessionInterface() 함수는 IOnlineSessionPtr 타입을 리턴하는데 받아주는 변수 OnlineSessionInterface는 IOnlineSessionPtr 타입으로 선언하면 안됐다.</br>
IOnlineSessionPtr이 typedef타입으로 선언되어 있어, <strong>TSharedPtr&lt;class IOnlineSession, ESPMode::ThreadSafe></strong> 타입으로 선언해줘야 했다.</br>

델리게이트는 모든 함수에 대한 레퍼런스를 갖는 오브젝트라고 생각할 수 있다.</br>
델리게이트는 함수를 바인딩하고, 바인딩 된 함수는 델리게이트가 브로드캐스트하는 것을 받고 응답할 수 있다.</br>
보통 콜백 함수를 델리게이트에 바인딩하고 게임에서 특정 사건이 발생하면 델리게이트가 Fired되거나 브로드캐스트되고 해당 델리게이트에 바인딩 된 함수들은 호출된다.</br></br>

CreateSession() 함수를 호출하면 정보가 stema으로 전송되고 게임 세션이 생성된다.</br>
스팀에서는 컴퓨터로 정보를 다시 보내주는데 게임 세션이 생성되었다는 알림을 보낸다.</br>
알림을 보낸다는 거에서 눈치를 챘을 수도 있지만, 세션 인터페이스는 델리게이트를 사용해야한다.</br>
델리게이트를 사용하기 위해 델리게이트 리스트를 만들어 적절한 이벤트에 실행해야한다.</br>

```
protected:
	UFUNCTION(BlueprintCallable)
	void CreateGameSession();

	void OnCreateSessionComplete(FName SessionName, bool bWasSuccessful);

private:

	FOnCreateSessionCompleteDelegate CreateSessionCompleteDelegate;
```

CreateGameSession은 게임 세션을 만드는 함수로 블루프린트에서 호출될 수 있도록 제작했다. 이렇게하면 BP_Player에서 키보드 이벤트에 해당 함수를 연결시키면 C++로 만든 함수가 실행된다.</br>
중요한 것은 밑에 코드로 델리게이트와 델리게이트와 바인딩 할 함수를 선언했다.</br>
OnCreateSessionComplete는 델리게이트와 바인딩 될 함수로 매개변수는 FOnCreateSessionCompleteDelegate에 맞춰서 작성했다.</br>
```
DECLARE_MULTICAST_DELEGATE_TwoParams(FOnCreateSessionComplete, FName, bool);
```
FOnCreateSessionCompleteDelegate의 정의로 이동하면 해당 델리게이트에 맞는 매개변수를 확인할 수 있다.</br>
델리게이트와 콜백 함수를 선언했으면 둘을 바인딩해줘야한다. 이는 Player 생성자에서 바인딩해준다.

```
AMenuSystemCharacter::AMenuSystemCharacter() : 
	CreateSessionCompleteDelegate(FOnCreateSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnCreateSessionComplete))
{
	...
}
```

CreateUObject는 클래스 멤버 함수를 델리게이트에 바인딩하는 데 사용되는 정적 메서드로 Player 객체를 넣고 바인딩할 함수를 적어주면 바인딩 절차가 끝나게 된다.</br>
여기서 ThisClass는 언리얼 엔진에서 제공하는 문법으로 간혹 현재 클래스 이름이 너무 길거나 난잡할때 ThisClass로 대체하여 작성할 수 있다.</br>

```
void AMenuSystemCharacter::CreateGameSession()
{
	if (!OnlineSessionInterface.IsValid())
	{
		return;
	}

	auto ExistingSession = OnlineSessionInterface->GetNamedSession(NAME_GameSession);
	if (ExistingSession != nullptr)
	{
		OnlineSessionInterface->DestroySession(NAME_GameSession);
	}

	// 델리게이트 리스트에 추가
	OnlineSessionInterface->AddOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegate);

	TSharedPtr<FOnlineSessionSettings> SessionSettings = MakeShareable(new FOnlineSessionSettings());
	SessionSettings->bIsLANMatch = false;
	.
	.
	.
	const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
	OnlineSessionInterface->CreateSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, *SessionSettings);

}
```

CreateGameSession 함수가 호출되면 세션 인터페이스가 있는지 확인하고 이미 만들어진 세션이 있는지 체크하여 있으면 기존 세션을 삭제해준다.</br>
델리게이트 리스트에 만든 델리게이트를 추가하여 세션 생성이 완료될 때 호출될 수 있도록한다.</br>
그 후 세션의 세팅을 설정해주고 플레이어의 고유 네트워크 ID를 가져와 세션 생성을 요청한다.</BR>

```
void AMenuSystemCharacter::OnCreateSessionComplete(FName SessionName, bool bWasSuccessful)
{
	if (bWasSuccessful)
	{
		if (GEngine)
		{
			GEngine->AddOnScreenDebugMessage(
				-1,
				15.f,
				FColor::Blue,
				FString::Printf(TEXT("Created session: %s"), *SessionName.ToString())
			);
		}
	}
 }
```

세션 생성이 완료되면 델리게이트를 통해 콜백 함수인 OnCreateSessionComplete함수가 호출된다. 함수가 하는 일은 간단한 디버그 메시지 출력이다.</br>

![createSession](https://github.com/user-attachments/assets/67cb84a1-c034-462c-8875-db9b8fb6e05b)

</br>
세션을 생성했으면 이제 해당 세션에 접속할 준비를 해야한다. 우선적으로 내가 원하는 세션을 찾을 필요가 있다.</br>
세션을 찾는 것도 세션 인터페이스에서 함수를 제공하며, 세션을 찾았다면 델리게이트를 통해 콜백 함수를 호출할 것이다.</br>
델리게이트 생성과 바인딩은 세션 생성과 거의 유사하니, 세션을 찾는 함수를 세세히 살펴보겠다.</br>

```
void AMenuSystemCharacter::JoinGameSession()
{
	// 게임 세션 찾기
	if (!OnlineSessionInterface)
		return;

	// 델리게이트 설정
	OnlineSessionInterface->AddOnFindSessionsCompleteDelegate_Handle(FindSessionCompleteDelegate);

	SessionSearch = MakeShareable(new FOnlineSessionSearch());
	SessionSearch->MaxSearchResults = 10000; 
	SessionSearch->bIsLanQuery = false;
	SessionSearch->QuerySettings.Set(SEARCH_PRESENCE, true, EOnlineComparisonOp::Equals);

	const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
	OnlineSessionInterface->FindSessions(*LocalPlayer->GetPreferredUniqueNetId(), SessionSearch.ToSharedRef());
}
```
델리게이트 설정까지 세션 생성 함수와 크게 다르지 않다. 중요하게 볼 것은 SessionSearch이다.</br>
세션 인터페이스의 함수 FindSessions를 실행하면 조건에 만족하는 세션을 얻어오는데 해당 세션들이 저장되는 곳이 SessionSearch이다.</br>
또한 검색할 세션들의 조건을 거는 것도 SessionSearch에다가 세팅을 해준다.</br>
우선 검색할 세션의 조건을 적고 세션 인터페이스의 함수 FindSessions에 입력해준다.</br>
여기서 ToSharedRef가 붙은 이유는 SessionSearh 변수의 타입이 TSharedPtr<FOnlineSessionSearch>이기 때문이다.</br>

```
void AMenuSystemCharacter::OnFindSessionComplete(bool bWasSuccessful)
{
	for (auto Result : SessionSearch->SearchResults)
	{
		FString Id = Result.GetSessionIdStr();
		FString User = Result.Session.OwningUserName;
		
		if (GEngine)
		{
			GEngine->AddOnScreenDebugMessage(
				-1,
				15.f,
				FColor::Cyan,
				FString::Printf(TEXT("Id: %s, User: %s"), *Id, *User)
			);
		}
	}
}
```

FindSession이 완료되면 델리게이트를 통해 콜백 함수인 OnFindSessionComplete가 호출된다.</br>
위에서 SessionSearch 변수가 찾은 세션들의 저장한다고 했는데, 세션들의 저장공간이 SearchResults 변수이다.</br>
SearchResults의 타입은 TArray<FOnlineSessionSearchResult>로 배열 타입이므로 for문을 통해 찾아낸 세션들을 하나씩 살펴본다.</br>


