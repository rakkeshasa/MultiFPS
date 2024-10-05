# MultiFPS(작업중)
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

CreateSession() 함수를 호출하면 정보가 steam으로 전송되고 게임 세션이 생성된다.</br>
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

![JoinSession](https://github.com/user-attachments/assets/ba5ce202-b65c-4593-89e3-9762d04b342b)

</br>

세션을 찾았으면 이제 해당 세션에 접속할 차례다.</br>
세션을 생성할 호스트를 위해 테스트용 맵인 Lobby를 하나 생성하고, Listen 서버로 서버를 열려고 한다.</br>
또한 세션에 접속을 했으면 델리게이트를 통해 콜백 함수를 호출할 예정이니 세션 접속용 델리게이트와 콜백 함수를 생성한다.</br>

```
void AMenuSystemCharacter::CreateGameSession()
{
	...
	SessionSettings->Set(FName("MatchType"), FString("FreeForAll"), EOnlineDataAdvertisementType::ViaOnlineServiceAndPing);
	...
}

void AMenuSystemCharacter::OnCreateSessionComplete(FName SessionName, bool bWasSuccessful)
{
	if (bWasSuccessful)
	{
		UWorld* World = GetWorld();
		if (World)
		{
			// 세션을 생성하자 마자 로비 맵으로 리슨 서버를 연다.
			World->ServerTravel(FString("/Game/ThirdPerson/Maps/Lobby.umap?listen"));
		}
	}
}
```

찾은 세션들 중 특정 세션과 연결하기 위해 매치 타입을 일치시켜야한다.</br>
그러기 위해서는 세션을 생성할 때 매치 타입을 지정해줘야하는데 CreateGameSession함수에서 매치 타입의 Key 값은 'MatchType'으로 Value 값은'FreeForAll' 로 정해줬다.</br>
세션 생성이 완료되면 델리게이트를 통해 OnCreateSessionComplete 콜백 함수가 호출되고 ServerTravel를 통해 Lobby 맵이 리슨 서버 형식으로 열리도록 한다.</br>

```
void AMenuSystemCharacter::OnFindSessionComplete(bool bWasSuccessful)
{
	for (auto Result : SessionSearch->SearchResults)
	{
		FString Id = Result.GetSessionIdStr();
		FString User = Result.Session.OwningUserName;
		FString MatchType;
		Result.Session.SessionSettings.Get(FName("MatchType"), MatchType);

		if (MatchType == FString("FreeForAll"))
		{
			OnlineSessionInterface->AddOnJoinSessionCompleteDelegate_Handle(JoinSessionCompleteDelegate);

			const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
			OnlineSessionInterface->JoinSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, Result);
		}
	}
}
```

세션 생성에 대한 수정이 완료됐다면 세션 찾는 것도 이제 매치 타입을 따져봐야한다.</br>
세션 검색을 하면 조건을 만족하는 세션들은 SessionResults 배열로 저장이 된다고 했다.</br>
배열에서 세션을 하나씩 살펴보면서 각 세션의 세팅을 확인해 매치 타입의 값이 어떻게 되는지 확인한다.</br>
세션 생성당시 SessionSettings의 Set을 이용해 매치 타입을 지정해줬고, 세션 검색시 Get을 통해 매치 타입의 값을 가져오고 있다.</br>
매치 타입의 값이 맞다면 세션에 입장할 차례다.</br.

```
void AMenuSystemCharacter::OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result)
{
	// IP 주소를 얻어 합류하기

	FString Address;
	if (OnlineSessionInterface->GetResolvedConnectString(NAME_GameSession, Address))
	{
		APlayerController* PlayerController = GetGameInstance()->GetFirstLocalPlayerController();
		if (PlayerController)
		{
			PlayerController->ClientTravel(Address, ETravelType::TRAVEL_Absolute);
		}
	}
}
```
세션에 접속하기 위해 호스트의 IP를 알아야한다.</BR>
호스트의 IP를 얻어오는 방법은 세션 인터페이스가 제공하는 함수를 이용하는 것이다.</BR>
GerResolvedConnectString을 통해 호스트의 IP주소를 Address 변수에 저장하고 ClientTravel을 통해 호스트 IP 세션에 접속할 수 있도록 한다.</BR>

![join](https://github.com/user-attachments/assets/7b09eed2-cbd5-4b23-b4c0-ff540675deb9)

</br>

## 버튼에 연동하기
Steam을 이용하여 여러 플레이어가 하나의 세션에 모여 같이 플레이할 수 있는 것을 확인했다.</br>
이제 세션 생성과 세션 참가를 버튼을 통해 할 수 있도록 할 것이다.</br>
안그러면 어느 플레이어가 세션을 생성할지(리슨 서버) 알 수 없기 때문이다.</br>
위에서는 언리얼에서 제공하는 플레이어 블루브린트에 1,2,3 키 이벤트를 사용하여 세션을 생성하고 참가했다.</br>
이번에는 게임에서 보이는 '게임 참가', '방 만들기'와 같은 버튼을 UI상에서 직접적으로 보여주기 위해 위젯 블루프린트를 사용했다.</BR></BR>

![wbp](https://github.com/user-attachments/assets/4ec34cd5-3ee6-49d5-98e2-46870d6872f5)
<div align="center"><strong>UI에 적용할 위젯 블루프린트</strong></div></BR>

</br>
위젯 블루프린트에 있는 버튼과 C++로 만든 세션관련 함수를 연동하기 위해 UserWidget을 상속받는 Menu 클래스를 생성했다.</br>

```
UCLASS()
class MULTIPLAYERSESSIONS_API UMenu : public UUserWidget
{
	GENERATED_BODY()

private:
	// BindWidget을 하면 WBP상의 버튼 위젯과 연결된다.
	UPROPERTY(meta = (BindWidget))
	class UButton* HostButton;

	UPROPERTY(meta = (BindWidget))
	UButton* JoinButton;

	UFUNCTION()
	void HostButtonClicked();

	UFUNCTION()
	void JoinButtonClicked();

	class UMultiplayerSessionsSubsystem* MultiplayerSessionsSubsystem;
};
```

재밌는 주의점은 위젯 블루프린트에서 생성한 버튼 이름과 Menu 클래스에서 생성한 버튼 이름이 같아야한다는 것이다.</br>
이름을 일치시키고 ```UPROPERTY(meta = (BindWidget))``` 를 통해 위젯 블루프린트의 버튼과 연결시킨다.</br></br>

<strong>HostButtonClicked</strong>와 <strong>JoinButtonClicked</strong>은 함수명에서 예측할 수 있듯이 버튼이 눌리면 델리게이트를 통해 호출될 함수이다.</br>
델리게이트와 연동하기 위해 ```UFUNCTION()``` 속성을 적용했다.</BR></BR>

```
bool UMenu::Initialize()
{
	if (!Super::Initialize())
	{
		return false;
	}

	// 버튼 클래스에 존재하는 델리게이트
	if (HostButton)
	{
		HostButton->OnClicked.AddDynamic(this, &ThisClass::HostButtonClicked);
	}
	if (JoinButton)
	{
		JoinButton->OnClicked.AddDynamic(this, &ThisClass::JoinButtonClicked);
	}

	return true;
}
```

Initialize 함수는 오버라이딩된 함수로 BeginPlay의 위젯 블루프린트 버전이다.</br>
버튼을 이용한 델리게이트 이벤트는 많은 게임에서 사용하고 버튼을 누르면 어디론가 신호를 보내야하기에 엔진에서 기본적으로 델리게이트에 관한 함수를 제공한다.
Button 클래스에 있는 OnClicked 함수는 버튼이 눌렸을 때 호출되는 함수로 AddDynamic을 통해 OnClicked가 호출되었을 때 콜백할 함수를 바인딩해준다.</br>

```
void UMenu::HostButtonClicked()
{
	HostButton->SetIsEnabled(false);
	if (MultiplayerSessionsSubsystem)
	{
		MultiplayerSessionsSubsystem->CreateSession(NumPublicConnections, MatchType);
	}
}

void UMenu::JoinButtonClicked()
{
	JoinButton->SetIsEnabled(false);
	if (MultiplayerSessionsSubsystem)
	{
		MultiplayerSessionsSubsystem->FindSession(10000);
	}
}
```

버튼에서 Onclicked 이벤트가 발생하면 콜백될 함수들은 OnlineSubsystem 챕터에서 봤던 세션 생성과 세션 찾기 함수를 호출한다.</br>
세션 관련 함수는 더 이상 플레이어 블루프린트에서 작동하지 않고 Menu 시스템에서 호출되도록 만들것이기 때문에 Character를 상속한 클래스에 있으면 안된다.</br></br>

세션관련 함수가 포함될 클래스는 더 이상 Character가 아닌 다른 무언가를 부모로 취해야한다.</br>
여기서 적합한 부모 클래스가 </strong>GameInstance 클래스</strong>이다.</br>
게임 인스턴스는 게임이 생성되고 게임이 종료될 때까지 소멸되지 않고, 다른 레벨 간에도 동일한 인스턴스가 유지되므로 부모로 적합한 클래스다.</br>
문제점은 게임 인스턴스는 넓은 영역을 관할는 엔진 클래스로 멀티플레이어 세션 기능이나, 캐릭터 클래스와 같은 다양한 클래스와 메소드를 포함한다는 것이다.</br>
이렇게 되면 해당 클래스를 컴파일 할때마다 프로그래밍 시간이 늘어나고, 엔진 클래스 오버라이드를 매번 체크해야한다는 단점이 있다.</br></br>

언리얼엔진에서는 이러한 단점을 해결하기 위해 Subsystem이라는 자동 인스턴싱 클래스를 만들었다.</br>
Subsystem 클래스는 쉬운 확정성을 제공하고, 블루프린트나 Python으로 클래스를 노출시켜 직관화가 가능하며 엔진 클래스의 오버라이드와 수정을 피할 수 있다.</br>
Subsystem의 종류는 Engine, Editor, GameInstance, LocalPlayer로 총 4가지가 있고 여기서 GameInstanceSubsystem을 사용할 것이다.</br></br>

둘의 연관성을 살펴보면</br>
1. 게임 인스턴스가 생성된 후, 게임 인스턴스 서브시스템도 생성된다.
2. 게임 인스턴스가 생성된 후 초기화 되면, 게임 인스턴스 서브시스템은 Initialize 함수를 호출해 초기화한다.
3. 게임 인스턴스가 종료되면, 게임 인스턴스 서브시스템은 Deinitialize 함수가 호출된다.
4. Deinitialize가 호출되면 게임 인스턴스 서브시스에 대한 참조가 삭제되고, 더 이상의 참조가 없으면 가비지 컬렉션된다.

상속받은 클래스 간의 생성자와 소멸자가 작동하는 원리와 비슷해보인다.</br>
이제 GameInstanceSubsystem을 통해 세션관련 함수를 관리할 것이다.</br>

```
UCLASS()
class MULTIPLAYERSESSIONS_API UMultiplayerSessionsSubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()
	
public:
	UMultiplayerSessionsSubsystem();

	void CreateSession(int32 NumPublicConnections, FString MatchType);
	void FindSession(int32 MaxSearchResults);
	void JoinSession(const FOnlineSessionSearchResult& SessionResult);
	void DestroySession();
	void StartSession();

	FMultiplayerOnCreateSessionComplete MultiplayerOnCreateSessionComplete;
	FMultiplayerOnFindSessionsComplete MultiplayerOnFindSessionsComplete;
	FMultiplayerOnJoinSessionComplete MultiplayerOnJoinSessionComplete;
	FMultiplayerOnDestroySessionComplete MultiplayerOnDestroySessionComplete;
	FMultiplayerOnStartSessionComplete MultiplayerOnStartSessionComplete;

protected:

	//
	// 델리게이트용 내부 콜백함수(세션 인터페이스 델리게이트 리스트)
	//
	void OnCreateSessionComplete(FName SessionName, bool bWasSuccessful);
	void OnFindSessionsComplete(bool bWasSuccessful);
	void OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result);
	void OnDestroySessionComplete(FName SessionName, bool bWasSuccessful);
	void OnStartSessionComplete(FName SessionName, bool bWasSuccessful);
};
```

세션 관련 함수들의 이동이 완료되었다.</br>
각 함수들의 내용은 Menu 클래스와 연동할 델리게이트 추가를 제외하면 MenuSystemCharacter 클래스에서 작성한 내용과 크게 다를바 없다.</br>
유심히 봐야할 것은 Menu 클래스가 어떻게 세션 관련 함수들로부터 응답을 받는지에 대한 것이다.</br>

```
void UMultiplayerSessionsSubsystem::CreateSession(int32 NumPublicConnections, FString MatchType)
{
	auto ExistingSession = SessionInterface->GetNamedSession(NAME_GameSession);
	if (ExistingSession != nullptr)
	{
		// 이미 세션이 존재한다면
		bCreateSessionOnDestroy = true;
		LastNumPublicConnections = NumPublicConnections;
		LastMatchType = MatchType;

		DestroySession();

		SessionInterface->DestroySession(NAME_GameSession);
	}

	// FDelegateHandle에 델리게이트 저장
	SessionInterface->AddOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegate);

	// 세션 세팅 관련 설정
	LastSessionSettings = MakeShareable(new FOnlineSessionSettings());
	.
	.
	.

	const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
	if (!SessionInterface->CreateSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, *LastSessionSettings))
	{
		SessionInterface->ClearOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegateHandle);
	
		// 커스텀 델리게이트에 브로드캐스트하기
		MultiplayerOnCreateSessionComplete.Broadcast(false);
	}

}
```
세션 생성 함수부터 살펴보면 기존 MenuCharacter 클래스의 함수와 다른 점은 이미 세션이 있는 경우에 또 세션 생성을 시도할 경우를 추가하였다.</br>
보통 세션이 정상적으로 종료되지 않고 세션을 재생성할때를 가정하여 기존 세션을 삭제하도록 했다.</br></br>

``` MultiplayerOnCreateSessionComplete.Broadcast(false) ``` 새로운 코드가 추가되었다.</br>
MultiplayerSessionSubsystem 클래스는 Session Interface의 세션 관련 함수를 호출하고 델리게이트를 통해 콜백 함수를 호출하지만,
Menu 클래스는 현재 ``` MultiplayerSessionSubsystem->CreateSession() ``` 을 해도 현재 정보를 얻어오지 못하고 있다.</br>
이를 해결하기 위해 MultiPlayerSessionSubsystem 클래스에 커스텀 델리게이트를 생성하고 메뉴 클래스에서 콜백 함수를 생성한다.</br>
즉 Menu 클래스가 CreateSession을 호출하면 MultiplayerSessionSubsystem 클래스가 Session Interface로 부터 응답을 받은 후 커스텀 델리게이트를 통해 Menu 클래스에도 응답을 보낸다.</br>
그러기 위해 커스텀 델리게이트인 MultiplayerOnCreateSessionComplete를 사용했다.</br></br>

커스텀 델리게이트 선언은 다음과 같다.</br>

```
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMultiplayerOnCreateSessionComplete, bool, bWasSuccessful);
DECLARE_MULTICAST_DELEGATE_TwoParams(FMultiplayerOnFindSessionsComplete, const TArray<FOnlineSessionSearchResult>& SessionResults, bool bWasSuccessful);
DECLARE_MULTICAST_DELEGATE_OneParam(FMultiplayerOnJoinSessionComplete, EOnJoinSessionCompleteResult::Type Result);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMultiplayerOnDestroySessionComplete, bool, bWasSuccessful);
```

매개변수 순서는 지정할 델리게이트 이름, 델리게이트에 바인딩할 변수 타입, 바인딩할 변수 이름이다.</br>
여기서 MULTICAST와 DYNAMIC 옵션을 확인 할 수 있는데 각각에 대해 설명하자면</BR>
MULTICAST : 브로드캐스트가 되면 다수의 클래스가 기능을 바인딩할 수 있다 </BR>
DYNAMIC : 델리게이트를 직렬화할 수 있고 블루프린트 그래프에서 로드될 수 있다 </BR>
이며 DYNAMIC의 경우 바인딩할 변수의 타입과 이름을 따로 따로 작성해야하며, 다이나믹 델리게이트와 바인딩하는 함수는 <STRONG>UFUCTION</STRONG>으로 선언된 함수여야하는 룰이 있다.</BR></BR>

이렇게 커스텀 델리게이트를 생성하고 CreateSession함수에서 해당 델리게이트를 통해 알맞은 값을 넣어 Menu 클래스에 통보해주면 된다.</br>
Menu 클래스에서는 OnCreateSession과 같은 콜백 함수를 생성해줬다.</br>

```
void UMenu::OnCreateSession(bool bWasSuccessful)
{
	if (bWasSuccessful)
	{
		UWorld* World = GetWorld();
		if (World)
		{
			World->ServerTravel(PathToLobby);
		}
	}
	else
	{
		HostButton->SetIsEnabled(true);
	}
}
```
콜백 함수인 OnCreateSession 함수가 할 일은 간단하다.</br>
델리게이트로 넘어온 값이 true면 ServerTravel을 통해 다른 레벨로 넘어가도록 해주고 false면 호스트 버튼은 재활성화하여 호스트 버튼을 다시 누를 수 있게 한다.</br></br>

커스텀 델리게이트도 생성하였고, 해당 델리게이트가 호출할 콜백 함수도 생성하였다.</br>
문제점은 Menu 클래스에 해당 델리게이트를 연동시키는 것인데 Menu 클래스가 세팅해야할 변수와 델리게이트를 하나의 함수에서 세팅할 수 있도록 새로운 함수를 생성했다.</br>

```
void UMenu::MenuSetup(int32 NumberOfPublicConnections, FString TypeOfMatch, FString LobbyPath)
{
	UGameInstance* GameInstance = GetGameInstance();
	if (GameInstance)
	{
		MultiplayerSessionsSubsystem = GameInstance->GetSubsystem<UMultiplayerSessionsSubsystem>();
	}

	if (MultiplayerSessionsSubsystem)
	{
		MultiplayerSessionsSubsystem->MultiplayerOnCreateSessionComplete.AddDynamic(this, &ThisClass::OnCreateSession);
		MultiplayerSessionsSubsystem->MultiplayerOnFindSessionsComplete.AddUObject(this, &ThisClass::OnFindSessions);
		MultiplayerSessionsSubsystem->MultiplayerOnJoinSessionComplete.AddUObject(this, &ThisClass::OnJoinSession);
		MultiplayerSessionsSubsystem->MultiplayerOnDestroySessionComplete.AddDynamic(this, &ThisClass::OnDestroySession);
	}
}
```

MultiPlayerSessionSubsystem 클래스에 커스텀 델리게이트를 생성했으므로 해당 클래스를 이용해야한다.</br>
MultiPlayerSessionSubsystem은 위에서 설명한거와 같이 GameInstance Subsystem을 부모로 갖는 클래스이고 GameInstance Subsystem은 GameInstance를 통해 가져올 수 있다.</br>
클래스를 가져왔다면 커스텀 델리게이트에 AddDynamic과 AddUObject를 통해 Menu 클래스에 있는 콜백함수와 바인딩하여 Menu 클래스가 세션 관련 함수로부터 알림을 받을 수 있게한다.</br></br>

커스텀 델리게이트가 true를 보내는 경우를 아직 살펴보지 못했다.</br>

```
void UMultiplayerSessionsSubsystem::OnCreateSessionComplete(FName SessionName, bool bWasSuccessful)
{
	if (SessionInterface)
	{
		SessionInterface->ClearOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegateHandle);
	}

	MultiplayerOnCreateSessionComplete.Broadcast(bWasSuccessful);
}
```

MultiplayerSessionSubsystem에서 Session Interface로 세션이 생성됐다는 알림을 받으면 OnCreateSessionComplete함수를 콜백하고,
핸들을 초기화하고 Menu 클래스한테 true값을 보내준다.</br></br>

CreateSession까지 완료했으니 이제는 FindSession을 살펴볼 차례이다.</br>
FindSession을 통해 찾은 세션들을 배열인 TArray에 넣고 이 배열을 델리게이트를 통해 Menu 클래스에 보내고자한다.</br>
MenuCharacter에서 FindSession을 다룰 때, FinSession을 하면 검색 조건에 맞는 세션을 SearchResults에 담아주며 이 변수는 TArray<FOnlineSessionSearchResult> 타입이라고 한 차례 설명한적이 있다.</br>
아쉽게도 SearchResults는 UCLASS나 UFUNCTION이 아니기 때문에 DYNAMIC 방식의 델리게이트를 사용하지 못한다.</BR>

```
DECLARE_MULTICAST_DELEGATE_TwoParams(FMultiplayerOnFindSessionsComplete, const TArray<FOnlineSessionSearchResult>& SessionResults, bool bWasSuccessful);
DECLARE_MULTICAST_DELEGATE_OneParam(FMultiplayerOnJoinSessionComplete, EOnJoinSessionCompleteResult::Type Result);

class MULTIPLAYERSESSIONS_API UMultiplayerSessionsSubsystem : public UGameInstanceSubsystem
{
	GENERATED_BODY()
	
public:
	FMultiplayerOnFindSessionsComplete MultiplayerOnFindSessionsComplete;
	FMultiplayerOnJoinSessionComplete MultiplayerOnJoinSessionComplete;
}
```

따로 USTRUCT를 만들어서 조건에 맞는 세션들을 저장해 델리게이트를 DYNAMIC하게 만들 수 있지만 SearchResults가 굳이 블루프린트에 노출될 필요가 없다고 느껴 DYNAMIC 속성을 제외하고 델리게이트를 정의했다.</BR>
JoinSession에서 얻는 EOnJoinSessionCompleteResult 또한 namespace일 뿐 언리얼엔진 구조체가 아닌 그냥 enum형 객체이므로 DYNAMIC을 적용할 수 없기에 속성을 제외하고 정의했다.</BR>
이후 MultiplayerSessionsSubsystem 클래스에서 Menu 클래스가 해당 델리게이트를 사용하기 위해 public 영역에 델리게이트를 선언해준다.</br>
그래야 Menu 클래스에서 MultiplayerSessionsSubsystem 클래스를 가져와서 델리게이트에 Menu 클래스의 콜백 함수를 바인딩 할 수 있기 때문이다.</br>

```
void UMenu::MenuSetup(int32 NumberOfPublicConnections, FString TypeOfMatch, FString LobbyPath)
{
	.
	.
	.

	if (MultiplayerSessionsSubsystem)
	{
		MultiplayerSessionsSubsystem->MultiplayerOnCreateSessionComplete.AddDynamic(this, &ThisClass::OnCreateSession);
		MultiplayerSessionsSubsystem->MultiplayerOnFindSessionsComplete.AddUObject(this, &ThisClass::OnFindSessions);
		MultiplayerSessionsSubsystem->MultiplayerOnJoinSessionComplete.AddUObject(this, &ThisClass::OnJoinSession);
	}
}

void UMenu::JoinButtonClicked()
{
	JoinButton->SetIsEnabled(false);
	if (MultiplayerSessionsSubsystem)
	{
		MultiplayerSessionsSubsystem->FindSession(10000);
	}
}
```

콜백 함수인 OnFindSessions함수는 DYNAMIC 델리게이트에 바인딩 되는것이 아니기 때문에 UFUNCTION()타입이 필요 없다.</BR>
또한 DYNAMIC 델리게이트가 아니므로 콜백함수와 바인딩을 할 때 AddUOject를 사용한다.</br></br>

Menu 클래스에서 Join 버튼이 눌리면 MultiplayerSessionsSubsystem 클래스의 FindSession 함수를 호출할 것이다.</br>
FindSession 함수는 Session Interface의 FindSessions을 통해 조건에 맞는 세션을 찾고 델리게이트를 통해 Menu 클래스에 응답해줄것이다.</br>

```
void UMultiplayerSessionsSubsystem::FindSession(int32 MaxSearchResults)
{
	FindSessionsCompleteDelegateHandle = SessionInterface->AddOnFindSessionsCompleteDelegate_Handle(FindSessionsCompleteDelegate);

	LastSessionSearch = MakeShareable(new FOnlineSessionSearch());
	LastSessionSearch->MaxSearchResults = MaxSearchResults;
	LastSessionSearch->bIsLanQuery = IOnlineSubsystem::Get()->GetSubsystemName() == "NULL" ? true : false;
	LastSessionSearch->QuerySettings.Set(SEARCH_PRESENCE, true, EOnlineComparisonOp::Equals);

	const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
	if (!SessionInterface->FindSessions(*LocalPlayer->GetPreferredUniqueNetId(), LastSessionSearch.ToSharedRef()))
	{
		SessionInterface->ClearOnFindSessionsCompleteDelegate_Handle(FindSessionsCompleteDelegateHandle);

		MultiplayerOnFindSessionsComplete.Broadcast(TArray<FOnlineSessionSearchResult>(), false);
	}
}
```

MenuCharacter 클래스에서 구현했던 기능과 크게 차이 없고, MultioplayerSessionsSubsystem 클래스에서 구현한 CreateSession과 비슷하게 작성되었다.</br>
세션 인터페이스에서 FindSessions를 실패하면 델리게이트 핸들을 삭제하고 빈 TArray와 세션 찾기에 실패했다는 의미의 false를 보낸다.</br>
FindSessions에 성공했다면 MultiplayerSessionsSubsystem 클래스의 콜백함수인 OnFindSessions가 호출될 것이다.</br>

```
void UMultiplayerSessionsSubsystem::OnFindSessionsComplete(bool bWasSuccessful)
{
	// 세션을 성공적으로 찾았을 시 델리게이트 삭제
	if (SessionInterface)
	{
		SessionInterface->ClearOnFindSessionsCompleteDelegate_Handle(FindSessionsCompleteDelegateHandle);
	}

	if (LastSessionSearch->SearchResults.Num() <= 0)
	{
		MultiplayerOnFindSessionsComplete.Broadcast(TArray<FOnlineSessionSearchResult>(), false);
		return;
	}

	MultiplayerOnFindSessionsComplete.Broadcast(LastSessionSearch->SearchResults, bWasSuccessful);
}
```
FindSessions에 성공했다면 델리게이트 핸들을 삭제하고 찾은 세션이 있는지 확인하기 위해 배열의 크기를 확인한다.</br>
아쉽게도 세션을 하나도 못 찾았다면 실패한것과 다름 없으므로 빈 배열과 false를 델리게이트를 통해 Menu 클래스에 보낸다.</br>
만약 세션을 1개 이상 찾았다면, SearchResults를 그대로 델리게이트에 실어서 Menu 클래스에 응답하게 된다.</br>

```
void UMenu::OnFindSessions(const TArray<FOnlineSessionSearchResult>& SessionResults, bool bWasSuccessful)
{
	for (auto Result : SessionResults)
	{
		FString SettingsValue;
		Result.Session.SessionSettings.Get(FName("MatchType"), SettingsValue);
		if (SettingsValue == MatchType)
		{
			MultiplayerSessionsSubsystem->JoinSession(Result);
			return;
		}
	}

	if (!bWasSuccessful || SessionResults.Num() == 0)
	{
		JoinButton->SetIsEnabled(true);
	}
}
```
Menu 클래스에서는 델리게이트를 통해 받은 응답으로 SessionResult의 내용물들을 하나씩 살펴보며 MatchType과 같은 세션이 있는지 확인하고 같다면 JoinSession을 호출하여 세션에 합류하는 단계로 넘어간다.</br>
만약 세션 찾기에 실패하면 Join 버튼이 다시 활성화 된다.</br></br>

JoinSession도 CreateSession과 FindSessions와 같은 흐름으로 진행되므로 따로 살펴보지 않고 Menu 클래스에서 어떻게 처리하는지만 살펴보겠다.</br>

```
void UMenu::OnJoinSession(EOnJoinSessionCompleteResult::Type Result)
{
	IOnlineSubsystem* Subsystem = IOnlineSubsystem::Get();
	if (Subsystem)
	{
		IOnlineSessionPtr SessionInterface = Subsystem->GetSessionInterface();
		if (SessionInterface.IsValid())
		{
			FString Address;
			SessionInterface->GetResolvedConnectString(NAME_GameSession, Address);

			APlayerController* PlayerController = GetGameInstance()->GetFirstLocalPlayerController();
			if (PlayerController)
			{
				PlayerController->ClientTravel(Address, ETravelType::TRAVEL_Absolute);
			}
		}
	}

	if (Result != EOnJoinSessionCompleteResult::Success)
	{
		JoinButton->SetIsEnabled(true);
	}
}
```

세션 인터페이스를 사용하기 위해 Online Subsystem이 필요하고, 이것은 MultiplayerSessionsSubsystem 클래스에 변수로 선언되어있지만 Private 영역에 있으므로 따로 얻어와야한다.</br>
Online Subsystem을 얻어온 후 세션 인터페이스를 가져오고, 세션 인터페이스를 통해 호스트의 IP 주소를 알아낸다.</BR>
Menu 클래스는 부모 클래스가 Pawn이나 Character가 아니기 때문에 바로 GetController을 할 수 없으므로 GameInstance로 부터 컨트롤러를 가져오고</br>
ClientTravel을 통해 호스트 세션으로 최종적으로 접속하게 된다.</br>


## 대기 레벨 만들기
일정 인원이 모이면 게임이 시작되며 인원이 모이기 전까지는 게임이 시작되지 않고 일정 레벨에 대기하도록 하고자 한다.</BR>
오버워치를 예를 들자면 캐릭터 픽창에서 다른 사람들을 기다리는 장면을 생각해볼 수 있다.</BR>

![OVERWATCH](https://github.com/user-attachments/assets/df2c8567-398a-4213-bcf8-ff9a321a03a8)
<div align="center"><strong>오버워치 게임에서 일정 인원이 채워질때까지 대기하는 화면</strong></div></BR>

이 게임은 캐릭터가 하나이므로 오버워치같이 거창한 UI나 캐릭터 풀이 없고 배틀그라운드 마냥 하나의 맵에서 그냥 뛰어다닐수 있게 할 것이다.</BR>
그러기 위해서는 우선 세션에 접속한 플레이어가 총 몇 명인지 파악할 수 있어야 한다.</BR></BR>

몇 명인지 파악하기 위해서는 Game Mode와 Game State를 사용할 것이다.</br>
게임 모드는 다른 레벨로 이동하거나 스폰 포인트 지정과 같은 게임의 모든 규칙을 세팅하는 것이고, 게임 스테이트는 게임 관련 정보를 저장하는 것이다.</br>
여기서 게임 모드는 클라이언트가 게임에 접속하면 ```PostLogin(APlayerController* NewPlayer)```가 호출되고, 게임을 종료하면 ```Logout(AController* Exiting)``` 을 호출한다.</br>
위 함수가 호출되면서 게임 스테이트는 PlayerState의 배열을 추가하거나 줄인다.</br>
게임 모드는 게임 스테이트에 접근할 수 있고, PlayerState의 크기를 확인하면서 세션에 플레이어가 몇 명 있는지 확인할 수 있다.</br></br>

대기 레벨에 적용할 게임 모드를 따로 생성하고 해당 게임 모드에서는 게임 스테이트에 있는 PlayerState의 배열의 크기를 확인할 것이다.</br>

```
void ALobbyGameMode::PostLogin(APlayerController* NewPlayer)
{
	Super::PostLogin(NewPlayer);

	int32 NumberOfPlayers = GameState.Get()->PlayerArray.Num();
	if (NumberOfPlayers == 2)
	{
		UWorld* World = GetWorld();
		if (World)
		{
			// 게임 모드는 서버에만 존재하고 서버에서 호출하는 것이 확실하다
			bUseSeamlessTravel = true; // 심리스 방식
			World->ServerTravel(FString("/Game/Maps/BlasterMap?listen"));
		}
	}
}
```
LobbyGameMode 클래스는 당연히 GameMode를 부모로 갖는 클래스로 PostLogin은 GameMode에 있는 함수로 오버라이딩됐다.</br>
게임 모드는 게임 스테이트에 접근할 수 있으므로 다른 객체를 통하거나 다른 클래스를 선언할 필요 없이 게임 스테이트에 접근하여 PlayerArray 배열의 크기를 NumberOfPlayers 변수에 담는다.</br>
일단 테스트환경이 열악하여 2명만 모이면 바로 대기 레벨에서 게임 레벨로 넘어가도록 했다.</br>
만약 2명이 모였다면 심리스 방식으로 게임 레벨로 플레이어가 옮겨가게 했다.</br></br>

```bUseSeamlessTravel = true``` 를 통해 어느정도 눈치 챘으리라 생각한다.</br>
<strong>심리스 방식</strong>은 플레이어가 다른 레벨로 이동할 때 서버와의 연결을 끊지 않고 다른 레벨로 이동하는 방식이다.</br>
만약 논심리스 방식으로 레벨을 이동한다면 클라이언트는 접속을 끊은 뒤 서버에 재연결하여 맵을 새로 로드해야하므로 게임이 멈추는 현상이 발생한다.</br>
그러면 모두 심리스 방식으로 하는게 좋아보이지만 현실적으로 논심리스 방식이 적용되는 곳도 있다.</br>
맵이 처음 로딩되거나, 서버에 처음 접속할 때, 새로운 게임을 실행할 때는 논심리스 방식이 적용된다.</br></br>

심리스 방식으로 레벨을 이동하기 위해서는 중간 레벨인 <strong>전환 레벨(Transition Level)</strong>이 필요하다.</br>
전환 레벨 없이 레벨을 이동하려면 항상 다른 레벨이 로드되어야 있어야하므로 메모리적으로 낭비이다.</br>
이를 위해서 중간 레벨인 전환 레벨을 통해 상대적으로 작은 레벨에서 대기하면서 넘어갈 다음 레벨을 로드하는 방식이다.</br>
RPG게임에서 오픈월드가 아닌 이상 볼 수 있는 로딩 창이 이러한 역할을 한다고 볼 수 있다.</BR></BR>

레벨을 넘어갈 방식을 심리스로 할지 논심리스로 할지 정했으면, 레벨 이동 함수를 어떤 것을 사용할지 정해야한다.</BR>
지금까지 ServerTravel 함수를 사용했지만 함수 이름에서 유추할 수 있듯이 서버상에서만 사용할 수 있는 함수이다.</br>
ServerTravel이 호출되면 서버는 다른 레벨로 넘어가게 되고 서버에 연결된 모든 클라이언트들은 서버가 APlayerController::ClientTravel를 호출함으로써 다른 레벨로 이동하게 된다.</br></br>

<strong>ClientTravel</strong>은 호출하는 객체에 따라 다른 일을 한다.</br>
클라이언트가 호출하면 클라이언트는 새 서버로 이동하게되며, 서버가 호출하는 경우에는 클라이언트들은 서버가 지정해준 레벨로 이동하게 된다.</br>

```
if (World)
{
	// 게임 모드는 서버에만 존재하고 서버에서 호출하는 것이 확실하다
	bUseSeamlessTravel = true; // 심리스 방식
	World->ServerTravel(FString("/Game/Maps/BlasterMap?listen"));
}
```
LobbyGameMode 클래스에 작성한 코드 중 핵심이라고 볼 수 있는 부분으로 다시 한번 보기 위해 코드를 가져왔다.</br>
게임 모드는 서버상에 있고 PostLogin 함수는 GameMode에서 제공하는 함수이므로 서버상에서 진행되는 것이 확실하므로 ServerTravel을 사용할 수 있다.</br></br>

정리하자면 게임 시작 화면에서 Host나 Join 버튼을 눌러 대기 레벨인 Lobby에서 다른 플레이어를 기다리고</br>
Lobby 레벨에서 플레이어가 2명이 모였다면 심리스 방식으로 잠시 전환 레벨에 머물렀다가 게임 레벨(Blaster)로 넘어가게 되는 방식이다.</br>



https://github.com/user-attachments/assets/51442161-3db6-414c-b0eb-8526fe89d804
<div align="center"><strong>시연 영상</strong></div></BR>

호스트 버튼을 눌러 Lobby 레벨로 이동하게 되고 열심히 뛰어다니는 동안 한 순간에 다른 플레이어가 보이면서 갑자기 화면이 까맣게 되는 것을 확인할 수 있다.</br>
이는 Lobby 레벨 블루프린트에서 현재 레벨에 있는 플레이어 수를 체크하고 Blaster 레벨로 이동하는데 심리스 방식으로 이동해서 중간 레벨인 까만 화면이 나오는 것이다.</br>
Blaster 맵으로 이동 후에는 정상적으로 플레이어가 맵을 돌아다니는 것을 볼 수 있다.</br></br>

## 총기 구현하기
우선 플레이어는 맵을 돌아다니면서 바닥에 있는 총기를 주워 장착하게 할 것이다.</br>
대부분의 게임에서 힐킷이나, 버프류 같은 것은 부딪히면 바로 적용되지만, 무기류는 덥석 집지 않고 따로 키를 조작하여 집게한다.</br>
이를 위해 플레이어가 바닥에 떨어진 총기에 가까이 갈 시 E키를 누르라는 위젯을 활성화 시킬 것이다.</BR>

![WeaponBP](https://github.com/user-attachments/assets/43bedb82-e864-4ddc-933f-bf8b9bd67218)
<div align="center"><strong>모든 총기의 부모 클래스가 될 BP_Weapon</strong></div></BR>

모든 총기의 토대가 될 BP_Weapon에는 총의 Mesh를 입힐 SkeletonMeshComponent인 Weapon Mesh와 플레이어와 충돌 처리를 할 SphereComponent인 Area Sphere, 설명문이 나올 위젯을 세팅할 WidgetComponent인 Pickup Widget가 포함되어 있다.</br>
충돌처리를 할 Area Sphere는 플레이어에게 반응 할 수 있도록 Collision을 따로 세팅해준다.</br>

이제 Area Sphere에 Overlap 이벤트를 발생시켜 위젯이 보이게 하면 되지 않나 싶지만, 이 게임은 멀티플레이게임이다.</br>
따라서 서버에서 플레이어가 총기에 충분히 접근했는지 판별하고 접근을 했다면 해당 플레이어를 조종중인 클라이언트에게 총기를 복제하여 위젯이 클라이언트 화면에 뜨게 해줘야한다.</br>

![Picture](https://github.com/user-attachments/assets/da0d3f87-cc52-490a-9915-4a6594b5c72d)
<div align="center"><strong>예시 그림</strong></div></BR>

즉 움직임이나 총기 줍기와 같은 행동은 서버에서 처리하고 처리한 결과를 클라이언트에게 전달하는 것이다.</br>
이를 위해서는 Overlap이벤트를 서버 환경에서만 실행시키도록 해야한다.</br>

```
void AWeapon::BeginPlay()
{
	Super::BeginPlay();
	
	if (HasAuthority())
	{
		AreaSphere->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
		AreaSphere->SetCollisionResponseToChannel(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Overlap);
		AreaSphere->OnComponentBeginOverlap.AddDynamic(this, &AWeapon::OnSphereOverlap);
		AreaSphere->OnComponentEndOverlap.AddDynamic(this, &AWeapon::OnSphereEndOverlap);
	}
}
```

<strong>HasAuthority</strong> 함수는 현재 서버 환경인가 체크하는 함수로 if문으로 감싸면 if문 안에 있는 코드는 서버 환경에서만 작동하게 된다.</br>
총기가 서버 환경인지 클라이언트 환경인지 어떻게 파악하는지 궁금할 수도 있으나 Listen 서버로 작동하는 이상 무조건 서버 환경에 있는 총은 1개가 존재한다.</br>
그렇다면 예상할 수 있듯이 <strong>충돌 처리를 서버 환경에서만 처리하므로</strong> 클라이언트가 총기와 부딪혀도 서버를 제외한 아무도 상호작용을 볼 수 없다.</br>
오직 서버에 있는 플레이어만 해당 상호작용을 볼 수 있으므로 이 상호작용을 알맞은 클라이언트에게 복제해서 전달해줘야한다.</br>
지금은 총기에 가까이 가면 BP_Weapon에 있는 WidgetComponent가 활성화 되어야 하므로 총기가 복제되어야한다.</br></br>

총기를 복제하기 위해서는 총기가 복제될 대상이라는 것을 알려야하고 복제된 대상인 총기를 소유할 클래스를 지정해줘야한다.</br>
총기는 당연히 플레이어가 소유하므로 BlasterCharacter 클래스가 복제된 총기를 가지게 되고, 이를 위해 지역 변수로 선언해준다.</br></br>

<strong>버그 리포트</strong>
HasAuthority는 생성자에서 작동을 하지 않는다.</br>

```
UPROPERTY(ReplicatedUsing = OnRep_OverlappingWeapon)
class AWeapon* OverlappingWeapon;

UFUNCTION()
void OnRep_OverlappingWeapon(AWeapon* LastWeapon);
```

헤더 파일에 작성된 코드이다.</br>
복제될 대상은 복제 속성을 반드시 가져야하므로 UPROPERTY(Replicated)가 작성되어야 한다.</br>
만약 복제되면서 호출할 함수가 있다면 해당 함수는 이름 앞에 <strong>'OnRep_'</strong>을 붙이고 속성을 <strong>UPROPERTY(ReplicatedUsing = 호출할 함수명)</strong>으로 해야한다.</br>
특히 호출될 함수는 어떤 매개변수도 갖지 못하나 예외적으로 복제되면서 자신을 호출할 클래스는 매개변수로 받을 수 있다. 여기서는 Weapon클래스가 매개변수로 들어갈 수 있다.</br>
여기까지는 변수에 그냥 '복제가능한' 옵션만 달린 것으로 제대로 활용하기 위해서는 따로 처리를 해줘야한다.</br></br>

```
void ABlasterCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION(ABlasterCharacter, OverlappingWeapon, COND_OwnerOnly);
}
```

Replicated 속성을 가진 변수를 활용하기 위해서는 GetLifetimeReplicatedProps 함수에 복제할 변수를 등록해줘야한다.</br>
DOREPLIFETIME_CONDITION( 복제된 변수를 갖는 클래스, 복제될 변수, 조건 )을 통해 복제될 변수인 Weapon을 등록해준다.</br>
여기서 조건인 <strong>COND_OwnerOnly</strong>이 핵심 포인트로 Owner는 자신의 기기에서 Pawn을 직접 조종하는 클라이언트를 의미한다.</br>
이를 통해 총기에 접근한 클라이언트만 Weapon 클래스인 OverlappingWeapon을 복제받을 수 있다.</br></br>

![rep](https://github.com/user-attachments/assets/ce3307c0-6fe4-43b0-882f-d00108ea05fc)
</br></br>


OverlappingWeapon은 플레이어가 총기와 충돌했을 때만 복제가 되어야한다.</br>
즉, Weapon클래스에 있는 OnSphereOverlap또는 OnSphereEndOverlap이 호출됐을 때만 복제가 되어야한다.</br>

```
void AWeapon::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	ABlasterCharacter* BlasterCharacter = Cast<ABlasterCharacter>(OtherActor);
	if (BlasterCharacter)
	{
		BlasterCharacter->SetOverlappingWeapon(this);
	}
}

void AWeapon::OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	ABlasterCharacter* BlasterCharacter = Cast<ABlasterCharacter>(OtherActor);
	if (BlasterCharacter)
	{
		BlasterCharacter->SetOverlappingWeapon(nullptr);
	}
}

void ABlasterCharacter::SetOverlappingWeapon(AWeapon* Weapon)
{	
	OverlappingWeapon = Weapon;
}
```
플레이어가 총기와 충돌했을 때 부딪힌 총기를 OverlappingWeapon에 세팅해주고, 충돌에서 벗어날 때 nullptr을 OverlappingWeapon에 세팅을 해주고 있다.</br>
충돌에서 벗얼날 때 nullptr를 보내는 이유는 총기에서 멀어졌을 때 위젯을 비활성화하기 위해 일종의 bool 변수와 비슷한 역할을 맡는다고 할 수 있다.</br>
이렇게하면 OverlappingWeapon 변수의 값이 변경될 때마다 총기와 부딪힌 플레이어를 조종하는(Owner) 클라이언트에게 복제가 된다.</br></br>


```
void ABlasterCharacter::OnRep_OverlappingWeapon(AWeapon* LastWeapon)
{
	if (OverlappingWeapon)
	{
		OverlappingWeapon->ShowPickupWidget(true);
	}

	if (LastWeapon)
	{
		// 저번에 복제된 무기가 있다면 위젯 숨기기
		LastWeapon->ShowPickupWidget(false);
	}
}
```
총기가 복제가 됐으면 이제 총기에 있는 위젯을 보여줄 차례다.</br>
복제된 총기로부터 함수가 호출되므로 이 함수는 클라이언트에서 작동되는 함수이다.</br>
잠시 함수 호출 흐름을 보자면 HasAuthority -> OnSphereOverlap -> SetOverlappingWeapon -> OnRep_OverlappingWeapon 순으로 호출이 된다.</br>
SetOverlappingWeapon까지는 서버환경에서 작동하고 <strong>총기가 서버에서 클라이언트로 복제되면서</strong> 복제된 총기가 호출하는 OnRep_OverlappingWeapon는 클라이언트에서 작동하게 된다.</br></br>

매개변수인 AWeapon* LastWeapon은 지금 복제된 무기가 아니라 바로 이전에 복제된 무기를 갖고 있다.</br>
만약에 내가 권총에 부딪히고 지나갔다면 SetOverlappingWeapon(권총)이 호출된 후 SetOverlappingWeapon(nullptr)이 호출되게 된다.</br>
처음에는 권총 이전에 복제된 무기가 없으므로 LastWeapon은 nullptr값을 갖고, OverlappingWeapon에는 권총이 세팅이 되어, 권총의 위젯이 활성화 된다.</br>
이후에는 LastWeapon이 권총이 되어 위젯이 비활성화 되고, OverlappingWeapon에는 nullptr이 들어가 ```if (OverlappingWeapon)``` 문을 통과한다.</br>
이전에 OnSphereEndOverlap에서 SetOverlappingWeapon(nullptr)을 하는 이유를 볼 수 있는 코드이다.</br></br>

이제 서버환경에서만 보이던 상호작용이 클라이언트에서 볼 수 있게 됐다. 하지만 문제점이 발생했다.</br>
OnRep_OverlappingWeapon가 클라이언트 환경에서만 작동하므로 ShowPickupWidget이 서버환경에서는 호출이 안된다는 점이다.</br>
서버에서만 보이던 문제점을 해결했더니 반대로 서버에서만 안보이게 되어 리슨 서버 플레이어는 위젯을 못 보게 되었다.</br>
서버 플레이어에게 위젯이 보이게 하기 위해서는 서버 환경에서 위젯이 보이도록 처리해야하고 충돌 처리 이후에 작동해야하므로 SetOverlappingWeapon함수에서 관련 코드를 작성하는게 제일 적당하다.</br>

```
void ABlasterCharacter::SetOverlappingWeapon(AWeapon* Weapon)
{
	// HasAuthority() -> OnSphereEndOverlap() -> SetOverlappingWeapon(nullptr)
	if (IsLocallyControlled())
	{
		if (OverlappingWeapon)
		{
			OverlappingWeapon->ShowPickupWidget(false);
		}
	}
	
	OverlappingWeapon = Weapon;

	// 내가 현재 Controll하고 있는 BlasterCharacter인가?
	// 서버 환경에서 조종하는 BlasterCharacter? -> 리슨 서버를 담당하는 유저의 캐릭터
	if (IsLocallyControlled())
	{
		if (OverlappingWeapon)
		{
			OverlappingWeapon->ShowPickupWidget(true);
		}
	}
}
```
수정된 SetOverlappingWeapon 함수 버전이다.</br>
기존에는 ``` OverlappingWeapon = Weapon; ``` 한 줄만 있었는데 위아래로 몇 줄의 코드가 추가 되었다.</br>
<strong>IsLocallyControlled</strong> 함수는 내가 현재 조종하고 있는 상태인가를 나타내는 Pawn 클래스 함수이다.</br>
여기서 <strong>SetOverlappingWeapon 함수는 서버 환경에서만 호출되므로</strong> IsLocallyControlled를 호출하면 리슨 서버에 있는 유저만 국한되게 된다.</br>
따라서 이 함수에서 ``` if (IsLocallyControlled()) ``` 는 내가 리슨 서버 유저인가를 확인하는 코드이다.</br>
리슨 서버 유저라면 OverlappingWeapon이 갱신되어 복제되기 전에 기존의 OverlappingWeapon의 위젯을 비활성화하고, 갱신을 한 후 복제된 이후에는 복제된 총기가 nullptr이 아니라면 위젯을 활성화하게 된다.</br></br>

![authority](https://github.com/user-attachments/assets/f5726e7c-bf8e-4d44-9d95-73661f7e6ae3)
</br></br>

이렇게하여 자신이 리슨 서버이든 클라이언트이든 총기에 다가가면 위젯이 활성화되고 멀어지면 비활성화되게 했다.</br>
![WeaponWidget](https://github.com/user-attachments/assets/3f4513cd-c99f-4c52-b822-794bc1578acf)
</br></br>

이제 E키를 눌러 총기를 주워 캐릭터에게 부착해줄 차례이다.</BR>
총을 장착하거나 쏘거나 피해를 입히는 기능을 BlasterPlayer클래스에 다 넣기에는 너무 방대하므로 이러한 역할을 담당해줄 ActorComponent클래스인 CombatComponent를 생성한다.</br>
전투기능을 담당하는 CombatComponent는 리슨 서버 유저든 클라이언트든 모든 캐릭터가 갖고있어야 하므로 복제 속성을 갖고 있어야한다.</br>
기존에 복제 속성을 처리한것과 차이점으로 <strong>컴포넌트는 복제 속성만 주고 복제 대상이라고 등록할 필요가 없다.</strong></br>
마지막으로 복제된 컴포넌트는 자신의 주인인 플레이어와 주운 무기를 따로 저장할 변수가 필요하다.</br>

```
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class BLASTER_API UCombatComponent : public UActorComponent
{
private:
	class ABlasterCharacter* Character;
	AWeapon* EquippedWeapon;
}
```

CombatComponent에 있는 멤버 변수 Character에 자신이 조종하는 BlasterCharacter가 들어가야하는데 이 부분을 BlasterCharacter클래스에서 처리해줘야한다.</br>
캐릭터가 만들어지고 월드에 소환되기 전에 이 컴포넌트에 캐릭터를 지정해줘야 소환 된 후에 전투기능이 알맞게 작동할 것이다.</BR>
<strong>PostInitializeComponents</strong> 함수가 그 시점에 CombatComponent에 Character 변수 값을 세팅해줄 수 있다.</br></br>

![lifecycle2](https://github.com/user-attachments/assets/f9c7b0ae-9bdd-48ad-878b-7cdf8ab1a375)
<div align="center"><strong>언리얼엔진 라이프 사이클</strong></div></BR>

라이프 사이클을 보면 Actor가 만들어지고 월드에 소환되기전에 호출되는 함수라는 것을 알 수 있다.</br>
이 함수에서 해줄 것은 간단하다. CombatComponent클래스의 Character변수에 자기 자신을 세팅해주는것이 끝이다.</br>

```
void ABlasterCharacter::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (Combat)
	{
		Combat->Character = this;
	}
}
```
</br>
CombatComponent클래스에 무기 장착을 담당할 EquipWeapon은 플레이어가 E키를 누르면 호출되는 함수로 주운 무기를 입력받고 무기를 스켈레톤 소켓에 부착할 것이다.</br>

```
void UCombatComponent::EquipWeapon(AWeapon* WeaponToEquip)
{
	if (Character == nullptr || WeaponToEquip == nullptr) return;

	EquippedWeapon = WeaponToEquip;
	EquippedWeapon->SetWeaponState(EWeaponState::EWS_Equipped);

	const USkeletalMeshSocket* HandSocket = Character->GetMesh()->GetSocketByName(FName("RightHandSocket"));
	if (HandSocket)
	{
		HandSocket->AttachActor(EquippedWeapon, Character->GetMesh());
	}

	ShowPickupWidget(false);
	AreaSphere->SetCollisionEnabled(ECollisionEnabled::NoCollision);
}
```
총을 소켓에 붙여주고 주운 무기는 더이상 위젯이 안보이도록 비활성화하며 충돌처리도 없도록 했다.</br></br>

멀티플레이의 특성을 생각하면 무기를 줍는 행동도 서버에서만 가능하다.</br>
핵 플레이를 방지하거나 내 환경에서는 총을 주웠는데 다른 사람 환경에서는 총이 안주운 상태면 안되기 때문이다.</br>

```
if (HasAuthority())
{
	Combat->EquipWeapon(OverlappingWeapon);
}
```

여기서 클라이언트는 총을 줍지 못하는 문제점이 생긴다.</br>
EquipWeapon은 핵 방지나 동기화 처리를 위해 서버에서 처리해야하지만, 클라이언트도 EquipWeapon함수를 사용해야 총을 장착할 수 있다.</br>
이러한 문제점을 해결할 수 있는 방안으로 <strong>원격 프로시저 호출(Remote Procedure Call, RPC)가</strong> 있다.</br>
RPC는 한 컴퓨터에서 다른 컴퓨터에 위치한 프로시저나 함수를 호출하는 프로그래밍 패러다임이다.</BR></BR>

![rpc](https://github.com/user-attachments/assets/56ed24f6-cb12-4587-9aac-06e201c8ef78)
</BR>

클라이언트는 RPC를 통해 서버 환경에서 EquipWeapon 함수를 실행하면 된다.</br>
다음은 언리얼 엔진 환경에서 RPC를 구축하는 코드이다.</BR>

```
UFUNCTION(Server, Reliable)
void ServerEquipButtonPressed();
```

``` UFUNCTION(Server, Reliable) ```에서 Server는 서버 환경에 있는 함수를 실행하고 Reliable은 신뢰성을 의미한다.</br>
만약 서버가 클라이언트에서만 작동하는 함수를 실행하고 싶다면 ``` UFUNCTION(Client, Reliable) ``` 를 작성하면 된다.</br>
신뢰성이 있는 RPC는 함수 실행을 보장한다. 처리 과정은 네트워크의 TCP와 비슷한 방식으로 패킷 전송에 실패하면 다시 RPC를 보낸다.</BR>
만약 비신뢰성이라면 한 컴퓨터에서 다른 컴퓨터로 정보를 보낼 때 RPC가 중단될 수도 있다.</BR>
이렇게 함수를 구현했다면 <strong>ServerEquipButtonPressed 함수는</strong> 클라이언트가 원격 프로시저 호출을 하면 <strong>서버 환경에서 작동한다.</strong></BR>

```
void ABlasterCharacter::ServerEquipButtonPressed_Implementation()
{
	if (Combat)
	{
		Combat->EquipWeapon(OverlappingWeapon);
	}
}
```

RPC 구현 함수는 이름 뒤에 반드시 _Implementation 접미사를 붙여야한다.</BR>
이렇게 구현하면 클라이언트는 서버 측으로 요청을 보내 서버 환경에서 무기를 줍게 된다.</BR></BR>

```
void ABlasterCharacter::EquipButtonPressed()
{
	if (Combat)
	{
		if (HasAuthority())
		{
			Combat->EquipWeapon(OverlappingWeapon);
		}
		else
		{
			ServerEquipButtonPressed();
		}
	}
}
```
정리하면 E키를 누르면 리슨 서버 유저면 바로 ComatComponent 클래스의 EquipWeapon 함수를 호출할 것이고</br>
클라이언트 유저면 RPC 함수인 ServerEquipButtonPressed 함수를 통해 서버 환경에서 EquipWeapon 함수를 호출한다.</br></br>

서버 환경에서 무기를 줍는 것은 좋지만 장착한 무기는 클라이언트에게 다시 복제하여 보내줘야한다.</BR>
호출 순서를 살펴보면 HasAuthority -> EquipWeapon 순으로 서버에서만 진행되므로 위젯과 콜리전 비활성화가 클라이언트 측에서 적용되지 않고 있다.</br>
따라서 CombatComponent 클래스의 멤버 변수인 EquippedWeapon을 복제 속성으로 만들어주고 복제 대상으로 등록해야한다.</br>
무기의 현재 상태를 나타내는 WeaponState도 복제 속성으로 만들어 클라이언트가 장착한 무기의 상태도 복제 될 수 있도록한다.</br>

```
void AWeapon::SetWeaponState(EWeaponState State)
{
	WeaponState = State;

	switch (WeaponState)
	{
	case EWeaponState::EWS_Equipped:
		ShowPickupWidget(false);
		AreaSphere->SetCollisionEnabled(ECollisionEnabled::NoCollision);
		break;
	}
}

void AWeapon::OnRep_WeaponState()
{
	switch (WeaponState)
	{
	case EWeaponState::EWS_Equipped:
		ShowPickupWidget(false); // 장착된 상태면 위젯 비활성화
		break;
	}
}
```
무기가 장착되면서 SetWeaponState를 통해 무기 상태가 Equipped가 될것이며 이것은 서버 환경에서 처리되는 일이다.</br>
서버 환경에서는 OnSphereOverlapped와 같은 함수로 각종 콜리전과 충돌 이벤트를 처리했으므로 SetWeaponState 함수에서 콜리전과 위젯을 비활성화 해준다.</br>
이후 무기 상태가 변경되면서 복제될 무기 상태는 클라이언트로 가게 되므로 충돌과 관련없는 콜리전을 빼고 위젯만 비활성화시켜 복제 된 총의 위젯이 더 이상 안보이게 한다.</br></br>

<strong>< 조준하기 ></strong></br>
총을 장착했다면 총을 조준하여 쏠 준비를 해야한다.</br>
총을 장착한 상태로 마우스 우클릭을 하면 bool 타입 bAiming 변수를 true로 해주고 우클릭에서 떼면 false로 해준다.</br>
그리고 이 bool 타입에 따라 조준하는 애니메이션을 틀어주면 되는데, 다른 클라이언트에서도 조준하는 애니메이션을 틀어줄려면 bAiming의 변수가 복제가 되야한다.</br>

```
void UCombatComponent::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME(UCombatComponent, bAiming);
}

void UCombatComponent::SetAiming(bool bIsAiming)
{
	bAiming = bIsAiming;
	
	ServerSetAiming(bIsAiming);
}

void UCombatComponent::ServerSetAiming_Implementation(bool bIsAiming)
{
	bAiming = bIsAiming;
}
```

당연히 서버에서 해당 행동을 판정하고 클라이언트에게 복제해야하므로 RPC를 사용하여 복제한다.</BR>
SetAiming에서 서버와 클라이언트를 상관안하고 RPC를 호출하고 있다.</BR>
클라이언트가 서버로 호출하는 RPC의 경우, 클라이언트에서 조작중인 캐릭터는 서버에서 함수를 실행하고</BR>
서버가 서버로 호출하는 RPC의 경우, 서버에서 조작중인 캐릭터 또한 서버에서 함수를 실행하기에 환경에 상관없이 RPC를 호출하고 있다.</BR>

## 총 쏘기
많은 FPS게임들은 화면에 조준점을 제공하고, 총을 쏘면 총알이 조준점을 향해 나간다.</BR>
화면상의 조준점을 대고 총을 쏘면 플레이어는 그 지역에 맞을거라 기대하고 게임에서는 발사체가 조준점을 기준으로 나아가도록 해야한다.</BR></BR>

![EX2](https://github.com/user-attachments/assets/1e64f181-af59-4430-aca2-3edcd66c1e67)
<div align="center"><strong>오버워치의 조준점</strong></div></BR></BR>

조준점은 화면 중앙으로 중앙을 항해 총을 쏠 수 있도록 하고, 조준점으로 발사체가 나가도록 해야한다.</BR>
그러기 위해서는 화면 상에서 중앙을 잡고 그 위치를 3D 세상인 World상에서의 좌표를 잡아야 발사체가 조준점을 향해 나갈 수 있다.</br>

```
void UCombatComponent::TraceUnderCrosshairs(FHitResult& TraceHitResult)
{
	FVector2D ViewportSize;
	if (GEngine && GEngine->GameViewport)
	{
		GEngine->GameViewport->GetViewportSize(ViewportSize);
	}

	FVector2D CrosshairLocation(ViewportSize.X / 2.f, ViewportSize.Y / 2.f);

	FVector CrosshairWorldPosition;
	FVector CrosshairWorldDirection;
	bool bScreenToWorld = UGameplayStatics::DeprojectScreenToWorld(
		UGameplayStatics::GetPlayerController(this, 0),
		CrosshairLocation,
		CrosshairWorldPosition,
		CrosshairWorldDirection
	);
}
```

화면의 중앙을 따오는 건 엔진에서 Viewport 사이즈를 얻어와 절반씩 나눠주면 쉽게 얻을 수 있다.</br>
그래픽스에서 3D 물체를 투영시켜서 2D 화면상에서 그렸다면, 다시 3D 월드의 좌표를 구하기 위해서는 반투영을 해줘야하는데 <strong>DeprojectScreenToWorld</strong>이 그 역할을 한다.</br>
CrosshairWorldPosition은 조준점의 위치를 나타내고, CrosshairWorldDirection은 조준점이 나아가는 방향을 나타낸다.</br>
2D에서 점을 3D 세상을 변환시키면 선이 되기때문에 위치와 방향이 결과로 나온다.</BR></BR>

![deproject](https://github.com/user-attachments/assets/49f94ea8-24a0-4fd4-a8f9-adb572c1e58e)
<div align="center"><strong>2D에서 3D로 변환 결과</strong></div></BR></BR>


CrosshairWorldPosition은 이제 총알이 나가는 첫 위치가 되고, 발사되는 위치로부터 방향 벡터인 CrosshairWorldDirection에 나아가는 거리를 곱해주면 총알의 궤적이 구해진다.</BR>
이 궤적을 라인 트레이싱하여 맞는 대상이 찾으면 된다.</BR>

```
FVector Start = CrosshairWorldPosition;
FVector End = Start + CrosshairWorldDirection * TRACE_LENGTH;

GetWorld()->LineTraceSingleByChannel(
	TraceHitResult,
	Start,
	End,
	ECollisionChannel::ECC_Visibility
);
```

총알이 나가는 궤적을 구했으니 총을 쏘는 행동을 통해 총알이 나가도록 해야한다.</BR>
좌클릭을 누르면 전투 담당 컴포넌트인 CombatComponent 클래스는 총기가 격발하는 애니메이션과 플레이어가 총 쏘는 애니메이션 몽타주를 출력한다.</br>

```
void UCombatComponent::FireButtonPressed(bool bPressed)
{
	bFireButtonPressed = bPressed;

	if (Character && bFireButtonPressed)
	{
		Character->PlayFireMontage(bAiming);
		EquippedWeapon->Fire();
	}
}
```

![shot1](https://github.com/user-attachments/assets/7765cc8f-13c5-4ae2-a8e7-72b352b755d8)
<div align="center"><strong>실행 결과</strong></div></BR></BR>

총기의 애니메이션과 플레이어의 애니메이션 몽타주가 싱글플레이에서는 잘 작동했지만 멀티플레이 환경에서 다른 유저들한테는 보여지지 않았다.</br>
이것은 총기의 애니메이션과 플레이어의 애니메이션 몽타주가 다른 플레이어들한테 복제되지 않았기 때문이다.</br></br>

두가지 해결 방안이 있는데, bFireButtonPressed 변수를 복제 속성을 달아 복제를 하여 true값일 때만 애니메이션을 출력하는 방법과 RPC를 이용하는 방법이 있다.</BR>
bFireButtonPressed 변수를 복제 속성으로 할경우 서버에서 true로 설정할 때, 클라이언트도 true값을 받게 되고 애니메이션 블루프린트에서 true일 때만 애니메이션을 실행하면된다.</br>
단점으로는 격발 애니메이션을 연속으로 출력해야할 때 무조건 bFireButtonPressed 변수가 값이 변해야하고 그러기 위해서는 마우스를 클릭했다 떼는 것을 반복하는 방법밖에 없다.</br>
</br>
자동화기와 같은 돌격소총이나 계속해서 불을 뿜어내는 화염방사기와 같은 무기를 위해서는 클릭 유지시 계속해서 격발 애니메이션이 나가도록 해야한다.</BR>
따라서 RPC를 이용해 FireButtonPressed 함수가 호출될 때마다 서버에서 격발 애니메이션을 재생하도록 처리하면된다.</br>

```
void UCombatComponent::FireButtonPressed(bool bPressed)
{
	bFireButtonPressed = bPressed;

	if (bFireButtonPressed)
	{
		ServerFire();
	}
}

void UCombatComponent::ServerFire_Implementation()
{
	if (Character)
	{
		Character->PlayFireMontage(bAiming);
		EquippedWeapon->Fire(HitTarget);
	}
}
```

이제 bFireButtonPressed가 true일 때마다 RPC를 통해 서버 환경에서 애니메이션을 재생하게 된다.</BR>
서버 환경에서 애니메이션을 재생했지만 다른 클라이언트들에게는 격발 장면이 보이지 않는다.</BR>
서버에서만 처리했기 때문에 총을 쏜 플레이어는 자신의 화면에서 애니메이션을 못 보고 오히려 리슨 서버 플레이어가 해당 애니메이션을 보는 상황이다.</BR>
총을 격발하는 장면은 모든 플레이어들에게 똑같이 보여야하므로 서버 환경이든 클라이언트 환경이든 애니메이션이 보여야한다.</BR>
따라서 서버 환경에서만 애니메이션을 재생하는 것이 아닌, 클라이언트 환경에서도 똑같이 재생을 해줘야한다.</BR></BR>

서버와 클라이언트 동시에 실행시키기 위해 <strong>MulticastRPC</strong>가 있다.</BR>
멀티캐스트 RPC는 서버와 클라이언트 두 환경에서 실행이 가능하게끔 호출을 한다.</BR> 
호출하는 주체에 따라 어느 환경에서 실행되는지 다르다.</BR>

![rpc](https://github.com/user-attachments/assets/cf603aeb-8a6f-46ab-90bd-bed37dbc6862)
<div align="center"><strong>RPC에 관한 표</strong></div></BR></BR>

클라이언트에서 호출하면 클라이언트에서만 실행되므로 소용이 없고, 서버에서 호출해야 서버와 클라이언트 모두 실행할 수 있다.</BR>
따라서 멀티캐스트 RPC를 적절하게 사용하기 위해서는 서버 환경임을 보장받아야한다.</BR>
HasAuthority를 통해 서버 환경일 때만 멀티캐스트 RPC를 하거나 이미 서버 환경인 ServerFire_Implementation에서 멀티캐스트 RPC를 하는 법도 있다.</BR>

```
void UCombatComponent::ServerFire_Implementation()
{
	MulticastFire();
}

void UCombatComponent::MulticastFire_Implementation()
{
	if (Character)
	{
		Character->PlayFireMontage(bAiming);
		EquippedWeapon->Fire(HitTarget);
	}
}
```

최적화를 생각하면 RPC를 적게 사용하는 HasAuthority를 사용하는 것이 좋으나, 명시성을 위해 이렇게 작성하였다.</br>
이제 좌클릭을 하면 서버와 클라이언트 모두 애니메이션을 재생하는 함수를 실행하게 된다.</br>

![shot2](https://github.com/user-attachments/assets/2a1ef534-0a15-4339-8330-a2a2e1767987)
<div align="center"><strong>모든 화면에 보이는 격발 장면</strong></div></BR></BR>

격발을 했다면 총알이 나갈 차례다.</br>
라인 트레이싱을 통해 부딪힌 대상의 위치와 총구의 위치를 통해 벡터를 구해줘야한다.</br>

```
void AProjectileWeapon::Fire(const FVector& HitTarget)
{
	FTransform SocketTransform = MuzzleFlashSocket->GetSocketTransform(GetWeaponMesh());
	FVector ToTarget = HitTarget - SocketTransform.GetLocation();
	FRotator TargetRotation = ToTarget.Rotation(); // 방향 벡터
	
	FActorSpawnParameters SpawnParams;
	SpawnParams.Owner = GetOwner();
	UWorld* World = GetWorld();
		
	if (World)
	{
		World->SpawnActor<AProjectile>(
			ProjectileClass,
			SocketTransform.GetLocation(),
			TargetRotation,
			SpawnParams);
	}
}
```

SocketTransform 변수는 총구의 위치를 나타내고, 매개변수인 HitTarget은 라인트레이싱의 결과인 TraceHitResult.ImpactPoint의 값을 갖고 있다.</br>
총구로부터 맞은 대상까지 총알을 날리기 위해서는 <strong>어디서부터 총알이 출발할지, 어느 방향으로 나갈지</strong> 구해야한다.</br>
총알의 시작 지점은 총구로 SocketTransform이며, 날아가는 방향은 총구의 위치와 맞은 대상의 위치를 통해 벡터를 구할 수 있으며 따로 방향만 추출하면 방향 벡터를 구할 수 있다.</br></br>

![vector](https://github.com/user-attachments/assets/4ec30984-473f-4b0e-a08f-83aa2d6beb09)
<div align="center"><strong>간단한 그림 예시</strong></div></BR></BR>


출발점과 방향을 구했다면 SpawnActor<AProjectile>을 통해 발사체 클래스 객체를 소환하면 총알이 발사가 된다.</br>
이후 블루프린트에서 적절한 이동 속도와 중력값을 주면 알맞은 방향으로 날아가게 되며 멀티플레이를 위해 이 코드는 서버에서만 작동이 되야한다.</br>
다른 클라이언트와의 동기화나 버그성 플레이를 방지하기 위해 서버에서 사격과 관련된 일들을 처리하고 클라이언트에게 복제를 해줘야한다.</br>
여기서 Weapon은 이미 ``` bReplicated = true ``` 를 통해 복제 속성을 갖고 있으므로 서버에서 관리중이므로 HasAuthority만 체크하면 된다.</br></br>

```
void AProjectileWeapon::Fire(const FVector& HitTarget)
{
	if (!HasAuthority()) return;

	.
	.
	.
		
	if (World)
	{
		World->SpawnActor<AProjectile>(
			ProjectileClass,
			SocketTransform.GetLocation(),
			TargetRotation,
			SpawnParams);
	}
}
```

서버에서만 총알을 내보내므로 클라이언트는 총알과 관련된 일을 알 수가 없어 총을 쏴도 클라이언트 화면에서는 총알이 안나가고 서버 화면에서만 총알이 나가게된다.</br>
또한 클라이언트 측에서 어딘가를 조준해서 쏴도 HitTarget이 복제되지 않아 서버 환경의 HitTarget에 총알이 나가게 된다.</br>
따라서 클라이언트도 HitTarget의 위치를 따로 서버에 보내고, Fire 함수를 클라이언트에서도 실행해야 총알이 올바른 방향으로 날아가는 것을 클라이언트 화면에서 볼 수 있다.</br>

![projectile1](https://github.com/user-attachments/assets/62a6461f-54c1-4a4c-b9a3-11039aabaa36)
<div align="center"><strong>클라이언트 조준점이 아닌 서버 조준점으로 날아가는 총알</strong></div></BR></BR>

우선은 서버환경에서 날아가는 총알을 보기 위해 총알 클래스인 Projectile의 생성자에 복제 속성을 추가하여 클라이언트에서도 일단 서버의 조준점으로 총알이 잘못 날라가는 것을 볼 수 있다.</br>
문제점은 클라이언트가 조준한 HitTarget의 위치를 서버가 모른다는 점으로 서버에게 따로 조준점을 알려줘야한다.</BR>
이 문제점은 위에서 봤던 격발 애니메이션이 클라이언트에게서 안보이고 서버에서만 보이던 점과 유사하며 해결법도 공유할 수 있다.</BR>

```
void UCombatComponent::FireButtonPressed(bool bPressed)
{
	bFireButtonPressed = bPressed;

	if (bFireButtonPressed)
	{
		FHitResult HitResult;
		TraceUnderCrosshairs(HitResult);
		ServerFire(HitResult.ImpactPoint);
	}


}
```

사격버튼이 눌렸을 때 HitResult의 값을 구하고 전에 구현한 RPC를 통해 서버측으로 값을 보내면된다.</BR>
입력 변수인 HitResult.ImpactPoint는 ``` const FVector_NetQuantize& ``` 타입으로 레퍼런스 값이며 <strong>NetQuantize</strong>은 Vector의 파생형으로 직렬화가 추가되어 네트워크 전송이 더 편한 버전이다.</br>
ServerFire로 인해 호출되는 MulticastFire과 Fire함수 또한 클라이언트 화면에서 알맞은 방향으로 총알이 나갈 수 있도록 같은 값을 받게 해준다.</br>

```
void UCombatComponent::ServerFire_Implementation(const FVector_NetQuantize& TraceHitTarget)
{
	MulticastFire(TraceHitTarget);
}

void UCombatComponent::MulticastFire_Implementation(const FVector_NetQuantize& TraceHitTarget)
{
	if (Character)
	{
		Character->PlayFireMontage(bAiming);
		EquippedWeapon->Fire(TraceHitTarget);
	}
}
```

이렇게 하여 서버측으로 클라이언트가 조준하는 위치를 보내므로 문제점이 해결됐다.</br>
다시 정리하자면 클라이언트 화면상으로 날아가는 총알이 안보이는 문제점은 Projectile 클래스의 생성자에다가 복제 속성을 추가함으로써 서버에서 날아가는 총알이 클라이언트에게 복제돼 보이도록했고</br>
클라이언트 조준점으로 날아가지 않는 문제점은 서버에 조준점의 위치를 보내므로써 해결했다.</br></br>

```
void AWeapon::Fire(const FVector& HitTarget)
{
	if (FireAnimation)
	{
		WeaponMesh->PlayAnimation(FireAnimation, false);
	}
}

void AProjectileWeapon::Fire(const FVector& HitTarget)
{
	if (!HasAuthority()) return;

	.
	.
	.
		
	if (World)
	{
		World->SpawnActor<AProjectile>(
			ProjectileClass,
			SocketTransform.GetLocation(),
			TargetRotation,
			SpawnParams);
	}
}
```

이렇게 하여 클라이언트는 격발 애니메이션만 출력하고 오직 서버에서만 총알을 소환하여 날아가게 했으며 총알은 복제되어 클라이언트에게도 보이게 된다.</br>

![projectile2](https://github.com/user-attachments/assets/dc0d29fb-6413-4d28-9cf7-cb9b4edf9fc0)
<div align="center"><strong>클라이언트 조준점으로 날아가는 총알</strong></div></BR></BR>


<strong><조준하기></strong></br>
보통 FPS에서 무기를 조준하면 멀리 있는 공간을 확대하여 볼 수 있게 되고 더 정밀하게 사격이 가능하다.</BR>
1인칭 FPS의 경우 스나이퍼 라이플에 주로 줌인 기능이 있고, 3인칭 FPS에서는 플레이어 등 뒤에 카메라가 있기에 대부분의 무기에 줌인기능을 제공한다.</BR>
FPS에서 더욱 정밀한 사격을 통해 적을 맞추는 것은 플레이어에게 만족감을 향상시키기 때문에 대부분 줌인 기능을 포함하고 있다.</BR></BR>

줌인 기능을 구현하기 위해서는 FOV(시야각)을 조절해줘야한다.</BR>
평상시의 시야각을 미리 변수에 저장해놓고 조준 키를 누르면 카메라가 지정한 시야각으로 보여줘야한다.</BR>
시야각 또한 무기마다 다르거나 무기 파츠를 장착하여 배율을 다르게 할 수 있어 Weapon 클래스에 조준 시 바뀔 시야각을 변수에 담아 각 무기마다 다른 시야각을 가질 수 있게 했다.</br>
조준 시마다 시야각을 조정해야하므로 조준 중인지 확인해야 할 필요가 있으며, 저번에 애니메이션 블루프린트에 사용하기 위해 전투 기능 담당인 CombatComponent 클래스에 복제속성으로 bAiming 변수를 만들었다.
따라서 매 틱마다 조준 상태인지 확인하고, 상태별로 알맞은 FOV를 카메라에 세팅을 해줄 것이다.</BR>

```
void UCombatComponent::BeginPlay()
{
	if (Character->GetFollowCamera())
	{
		DefaultFOV = Character->GetFollowCamera()->FieldOfView;
		CurrentFOV = DefaultFOV;
	}
}

void UCombatComponent::InterpFOV(float DeltaTime)
{
	if (bAiming)
		CurrentFOV = FMath::FInterpTo(CurrentFOV, EquippedWeapon->GetZoomedFOV(), DeltaTime, EquippedWeapon->GetZoomInterpSpeed());
	else
		CurrentFOV = FMath::FInterpTo(CurrentFOV, DefaultFOV, DeltaTime, ZoomInterpSpeed);

	Character->GetFollowCamera()->SetFieldOfView(CurrentFOV);
}
```

줌 아웃을 할 때는 기존 FOV값을 가져와야하기 때문에 DefaultFOV에 따로 저장을 해놓는다.</BR>
매 틱마다 조준 중인지 확인하기 위해 TickComponent는 InterpFOV를 매 틱마다 호출하여 InterpFOV에서 매 틱마다 체크하게된다.</br>
FInterpTo 함수를 통해 현재 FOV에서 얼마만큼의 속도로 변화할지 정하고 SetFieldOfView로 카메라의 FOV를 세팅해줬다.</BR></BR>

![zoomout](https://github.com/user-attachments/assets/c942fc98-d941-4b42-aab5-d02c2d5317f5)
<div align="center"><strong>조준 해제 결과(평상시)</strong></div></BR></BR>

![ZoomIn](https://github.com/user-attachments/assets/12632753-2bc8-40d8-8e6e-fa125014a1c0)
<div align="center"><strong>조준 결과</strong></div></BR></BR>

<strong><피격효과></strong>
총알을 발사하여 캐릭터가 맞았으면 캐릭터는 피격 애니메이션을 취한다.</br>
피격 처리 또한 서버에서 처리하고 서버에 접속중인 모든 클라이언트에 복제하여 결과를 공유해 줘야한다.</br>
리슨 서버 환경에 있는 플레이어도 다른 모든 클라이언트에게도 애니메이션을 재생해야하므로 멀티캐스트 NPC가 적절한 방법이다.</BR>

```
void AProjectile::BeginPlay()
{
	Super::BeginPlay();

	if (HasAuthority())
	{
		CollisionBox->OnComponentHit.AddDynamic(this, &AProjectile::OnHit);
	}
}

void AProjectile::OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	ABlasterCharacter* BlasterCharacter = Cast<ABlasterCharacter>(OtherActor);

	if (BlasterCharacter)
	{
		BlasterCharacter->MulticastHit();
	}
	Destroy();
}

void ABlasterCharacter::MulticastHit_Implementation()
{
	PlayHitReactMontage();
}
```

총알을 담당하는 Projectile 클래스는 BeginPlay에서 HasAuthority를 체크해 서버 환경에서만 총알이 충돌 처리를 하도록 하여 OnHit이 서버 환경에서만 호출되는 것을 확인할 수 있다.</br>
OnHit 함수는 피격 대상을 확인하고 피격 대상의 멀티캐스트 RPC인 MulticastHit 함수를 호출한다.</br>
서버 환경에서 멀티캐스트 RPC를 호출했으므로 서버에 있는 플레이어와 모든 클라이언트의 플레이어는 MulticastHit함수의 내용을 실행하게 된다.</br>
PlayHitReactMontage 함수가 모든 컴퓨터의 플레이어에게서 실행되므로 피격된 플레이어의 피격 애니메이션이 보이게 된다.</br></br>

![Hit](https://github.com/user-attachments/assets/2e5bffa8-53fc-4207-a15e-c41e9e3cfac4)
<div align="center"><strong>모든 플레이어에게 공유되는 피격효과</strong></div></BR></BR>

<strong><자동 화기 구현하기></strong>
총기를 사격할 때 사격 버튼을 누르는 것을 복제 속성으로 처리할지 아니면 RPC로 처리할지 위에서 고민했으며, 자동화기의 경우 복제 속성으로 구현하기에는 어울리지 않아 RPC로 사격 처리를 하였다.</BR>
매 틱마다 사격 클릭이 눌렸는지 체크하고 눌렸으면 멀티캐스트 RPC를 통해 총을 발사하고 총알을 발사하는 애니메이션을 재생하도록 했다.</BR>
자동화기를 구현하기 위해서는 사격 처리를 하는 함수를 매번 불러 사격 애니메이션과 총에서 총알이 나가도록 해줘야한다.</br>
또한 사격 간의 시간을 두어 연사 속도를 조절할 수 있어야 하며 일정 시간이 되면 총알이 또 나가도록 해야하고 그 전까지는 총알이 절대 나가서는 안된다.</br>

```
void UCombatComponent::Fire()
{
	if (bCanFire)
	{
		bCanFire = false;
		ServerFire(HitTarget);
		StartFireTimer();
	}
}

void UCombatComponent::StartFireTimer()
{
	Character->GetWorldTimerManager().SetTimer(
		FireTimer,
		this,
		&UCombatComponent::FireTimerFinished,
		EquippedWeapon->FireDelay
	);
}

void UCombatComponent::FireTimerFinished()
{
	bCanFire = true;

	if (bFireButtonPressed && EquippedWeapon->bAutomatic)
	{
		Fire();
	}
}
```

여기서 Fire함수는 사격 클릭과 연동된 함수이며, ServerFire는 멀티캐스트 RPC를 호출하여 사격 애니메이션을 출력하고 총알이 나가도록 하는 역할을 한다.</BR>
bCanFire 변수는 일정 시간이 되면 true값을 가져 총알이 나가도록 하고 총알이 나가면 바로 false값이 되어 일정 시간이 지나지 않으면 총을 쏘지 못한다.</br>
따라서 bCanFire 변수는 총알을 쏠 수 있는지 제어해주는 역할을 하게 되며 초기화 시 무조건 true값을 가져야 첫 발이 나갈 수 있다.</br>
StartFireTimer은 FireDelay로 지정해둔 시간만큼 지난다면 FireTimerFinished를 호출하고, FireTimerFinished는 bCanFire을 true값으로 세팅하여 총을 쏠 수 있게 한다.</br>
일정 시간이 지난 뒤에 다시 호출 된 Fire 함수는 bCanFire값을 확인하고 true면 멀티캐스트 RPC를 통해 모든 플레이어에게 총을 쏘는 장면을 출력해준다.</BR>
이후 bCanFire은 false값을 가져 사격 클릭을 통해 호출되는 Fire가 실행되도 일정시간 동안 총을 쏘지 못하게 된다.</BR></BR>

코드에서 ``` EquippedWeapon->FireDelay ``` 와 ``` EquippedWeapon->bAutomatic ```를 볼 수 있다.</BR>
총기마다 연사 속도와 자동화기인지 지정할 수 있게 하여 스나이퍼 라이플은 연사가 안되게하거나 샷건은 연사 속도를 느리게하는것처럼 총마다 차별점을 두었다.</BR></BR>

