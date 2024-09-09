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
이며 DYNAMIC의 경우 바인딩할 변수의 타입과 이름을 따로 따로 작성해야하며, 다이나믹 델리게이트와 바인딩하는 함수는 UFUCTION으로 선언된 함수여야하는 룰이 있다.</BR></BR>

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
핸들을 초기화하고 Menu 클래스한테 true값을 보내준다.</br>
