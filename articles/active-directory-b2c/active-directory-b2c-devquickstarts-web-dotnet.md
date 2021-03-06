---
title: Azure Active Directory B2C | Microsoft Docs
description: Azure Active Directory B2C를 사용하여 로그인, 등록 및 프로필 관리를 포함하는 웹 응용 프로그램을 빌드하는 방법입니다.
services: active-directory-b2c
documentationcenter: .net
author: dstrockis
manager: msmbaldwin
editor: ''

ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 07/22/2016
ms.author: dastrock

---
# Azure AD B2C: .NET 웹앱 빌드
<!-- TODO [AZURE.INCLUDE [active-directory-b2c-devquickstarts-web-switcher](../../includes/active-directory-b2c-devquickstarts-web-switcher.md)]-->

Azure AD(Azure Active Directory) B2C를 사용하여 몇 가지 간단한 단계에서 강력한 셀프 서비스 ID 관리 기능을 웹앱에 추가할 수 있습니다. 이 문서에서는 사용자 등록, 로그인 및 프로필 관리를 포함하는 .NET Model-View-Controller(MVC) 웹앱을 만드는 방법을 보여 줍니다. 이 앱은 사용자 이름 또는 전자 메일을 사용하고 Facebook 및 Google과 같은 소셜 계정을 사용하여 등록 및 로그인에 대한 지원을 포함합니다.

## Azure AD B2C 디렉터리 가져오기
Azure AD B2C를 사용하기 전에 디렉터리 또는 테넌트를 만들어야 합니다. 디렉터리는 모든 사용자, 앱, 그룹 등을 위한 컨테이너입니다. 아직 없는 경우 [B2C 디렉터리를 만든](active-directory-b2c-get-started.md) 후에 이 가이드를 계속 진행합니다.

## 응용 프로그램 만들기
다음으로 B2C 디렉터리에서 앱을 만들어야 합니다. 앱과 안전하게 통신하는 데 필요한 Azure AD 정보를 제공합니다. 앱을 만들려면 [다음 지침](active-directory-b2c-app-registration.md)에 따릅니다. 다음을 수행해야 합니다.

* 응용 프로그램에서 **웹앱/웹 API**를 포함합니다.
* `https://localhost:44316/`을 **리디렉션 URI**으로 입력합니다. 이 코드 샘플에 대한 기본 URL입니다.
* 앱에 할당된 **응용 프로그램 ID**를 적복사합니다. 이 시간은 나중에 필요합니다.

[!INCLUDE [active-directory-b2c-devquickstarts-v2-apps](../../includes/active-directory-b2c-devquickstarts-v2-apps.md)]

## 정책 만들기
Azure AD B2C에서 모든 사용자 환경은 [정책](active-directory-b2c-reference-policies.md)에 의해 정의됩니다. 이 코드 샘플은 등록, 로그인 및 프로필 편집 등 세 가지 ID 환경을 포함합니다. [정책 참조 문서](active-directory-b2c-reference-policies.md#how-to-create-a-sign-up-policy)에서 설명한 대로 각 형식에 하나의 정책을 만들어야 합니다.

> [!NOTE]
> 또한 Azure AD B2C는 이 자습서에서는 자세히 다루지 않는 통합된 등록 또는 로그인 정책을 지원합니다. 등록 또는 로그인 정책은 [이 해당 자습서](active-directory-b2c-devquickstarts-web-dotnet-susi.md)에 제공됩니다.
> 
> 

세 가지 정책을 만들 때 다음을 확인합니다.

* ID 공급자 블레이드에서 **사용자 ID 등록** 또는 **전자 메일 등록**을 선택합니다.
* 등록 정책에서 **표시 이름** 및 다른 등록 특성을 선택합니다.
* 모든 정책에서 **표시 이름** 클레임을 응용 프로그램 클레임으로 선택합니다. 물론 다른 클레임을 선택할 수 있습니다.
* 각 정책을 만든 후에 **이름**을 복사합니다. 이러한 정책 이름이 나중에 필요합니다.

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

세 가지 정책을 만들었다면 앱을 빌드할 준비가 되었습니다.

## 코드 다운로드 및 인증 구성
이 샘플에 대한 코드는 [GitHub에서 유지 관리](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIdConnect-DotNet)됩니다. 진행하면서 샘플을 작성하기 위해 [골격 프로젝트를 .zip 파일로 다운로드](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIdConnect-DotNet/archive/skeleton.zip)할 수 있습니다. 구조를 복제할 수도 있습니다.

```
git clone --branch skeleton https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIdConnect-DotNet.git
```

완성된 샘플도 [.zip 파일로 다운로드하거나](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIdConnect-DotNet/archive/complete.zip) 동일한 리포지토리의 `complete` 분기에서 사용할 수 있습니다.

샘플 코드를 다운로드한 후 Visual Studio .sln 파일을 열어 시작합니다.

앱은 인증 메시지를 전송하여 Azure AD B2C와 통신하며 이는 HTTP 요청의 일부로 실행하고자 하는 정책을 지정합니다. .NET 웹 응용 프로그램의 경우 Microsoft의 OWIN 미들웨어를 사용하여 OpenID Connect 인증 요청, 정책 실행, 사용자의 세션 관리 등을 보낼 수 있습니다.

시작하려면 Visual Studio 패키지 관리자 콘솔을 사용하여 OWIN 미들웨어 NuGet 패키지를 프로젝트에 추가합니다.

```
PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
PM> Install-Package Microsoft.Owin.Security.Cookies
PM> Install-Package Microsoft.Owin.Host.SystemWeb
```

다음으로 프로젝트 루트에 있는 `web.config` 파일을 열고 `<appSettings>` 섹션에 앱의 구성 값을 입력하고 기존 값을 바꿉니다.

```
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:Tenant" value="fabrikamb2c.onmicrosoft.com" />
    <add key="ida:ClientId" value="90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6" />
    <add key="ida:AadInstance" value="https://login.microsoftonline.com/{0}/v2.0/.well-known/openid-configuration?p={1}" />
    <add key="ida:RedirectUri" value="https://localhost:44316/" />
    <add key="ida:SignUpPolicyId" value="b2c_1_sign_up" />
    <add key="ida:SignInPolicyId" value="b2c_1_sign_in" />
    <add key="ida:UserProfilePolicyId" value="b2c_1_edit_profile" />
  </appSettings>
...
```

[!INCLUDE [active-directory-b2c-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]

다음으로 `Startup.cs`라는 프로젝트에 OWIN 시작 클래스를 추가합니다. 프로젝트를 마우스 오른쪽 단추로 클릭하고 **추가** 및 **새 항목**을 선택한 다음 OWIN을 검색합니다. **클래스 선언을 `public partial class Startup`으로 변경해야 합니다**. 다른 파일에서 이 클래스의 일부를 구현했습니다. OWIN 미들웨어는 앱이 시작되면 `Configuration(...)` 메서드를 호출합니다. 이 메서드에서 `ConfigureAuth(...)`를 호출하여 앱에 대한 인증을 설정합니다.

```C#
// Startup.cs

public partial class Startup
{
    public void Configuration(IAppBuilder app)
    {
        ConfigureAuth(app);
    }
}
```

`App_Start\Startup.Auth.cs` 파일을 열고 `ConfigureAuth(...)` 메서드를 구현합니다. `OpenIdConnectAuthenticationOptions`에 제공하는 매개 변수는 앱이 Azure AD와 통신하기 위한 좌표로 사용됩니다. 쿠키 인증도 설정해야 합니다. 무엇보다도 OpenID Connect 미들웨어는 사용자 세션을 유지하기 위해 쿠키를 사용합니다.

```C#
// App_Start\Startup.Auth.cs

public partial class Startup
{
    // App config settings
    private static string clientId = ConfigurationManager.AppSettings["ida:ClientId"];
    private static string aadInstance = ConfigurationManager.AppSettings["ida:AadInstance"];
    private static string tenant = ConfigurationManager.AppSettings["ida:Tenant"];
    private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];

    // B2C policy identifiers
    public static string SignUpPolicyId = ConfigurationManager.AppSettings["ida:SignUpPolicyId"];
    public static string SignInPolicyId = ConfigurationManager.AppSettings["ida:SignInPolicyId"];
    public static string ProfilePolicyId = ConfigurationManager.AppSettings["ida:UserProfilePolicyId"];

    public void ConfigureAuth(IAppBuilder app)
    {
        app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

        app.UseCookieAuthentication(new CookieAuthenticationOptions());

        // Configure OpenID Connect middleware for each policy
        app.UseOpenIdConnectAuthentication(CreateOptionsFromPolicy(SignUpPolicyId));
        app.UseOpenIdConnectAuthentication(CreateOptionsFromPolicy(ProfilePolicyId));
        app.UseOpenIdConnectAuthentication(CreateOptionsFromPolicy(SignInPolicyId));
    }

    // Used for avoiding yellow-screen-of-death
    private Task AuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> notification)
    {
        notification.HandleResponse();
        if (notification.Exception.Message == "access_denied")
        {
            notification.Response.Redirect("/");
        }
        else
        {
            notification.Response.Redirect("/Home/Error?message=" + notification.Exception.Message);
        }

        return Task.FromResult(0);
    }

    private OpenIdConnectAuthenticationOptions CreateOptionsFromPolicy(string policy)
    {
        return new OpenIdConnectAuthenticationOptions
        {
            // For each policy, give OWIN the policy-specific metadata address, and
            // set the authentication type to the id of the policy
            MetadataAddress = String.Format(aadInstance, tenant, policy),
            AuthenticationType = policy,

            // These are standard OpenID Connect parameters, with values pulled from web.config
            ClientId = clientId,
            RedirectUri = redirectUri,
            PostLogoutRedirectUri = redirectUri,
            Notifications = new OpenIdConnectAuthenticationNotifications
            {
                AuthenticationFailed = AuthenticationFailed,
            },
            Scope = "openid",
            ResponseType = "id_token",

            // This piece is optional - it is used for displaying the user's name in the navigation bar.
            TokenValidationParameters = new TokenValidationParameters
            {
                NameClaimType = "name",
            },
        };
    }
}
```

## Azure AD로 인증 요청 보내기
이제 앱은 OpenID Connect 인증 프로토콜을 사용하여 Azure AD B2C와 통신하도록 올바르게 구성되었습니다. OWIN이 인증 메시지를 작성하고, Azure AD에서 토큰의 유효성을 검사하고, 사용자 세션을 유지 관리하는 모든 세부 과정을 처리했습니다. 이제 각 사용자 흐름을 시작하기만 하면 됩니다.

사용자가 웹앱에서 **등록**, **로그인** 또는 **프로필 편집**을 선택하는 경우 연결된 작업이 `Controllers\AccountController.cs`에서 호출됩니다. 각각의 경우 기본 OWIN 메서드를 사용하여 올바른 정책을 트리거할 수 있습니다.

```C#
// Controllers\AccountController.cs

public void SignIn()
{
    if (!Request.IsAuthenticated)
    {
        // To execute a policy, you simply need to trigger an OWIN challenge.
        // You can indicate which policy to use by specifying the policy id as the AuthenticationType
        HttpContext.GetOwinContext().Authentication.Challenge(
            new AuthenticationProperties () { RedirectUri = "/" }, Startup.SignInPolicyId);
    }
}

public void SignUp()
{
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(
            new AuthenticationProperties() { RedirectUri = "/" }, Startup.SignUpPolicyId);
    }
}


public void Profile()
{
    if (Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(
            new AuthenticationProperties() { RedirectUri = "/" }, Startup.ProfilePolicyId);
    }
}
```

컨트롤러에서 `[Authorize]` 태그를 사용하여 사용자가 로그인하지 않은 경우 특정 정책이 실행되도록 할 수도 있습니다. `Controllers\HomeController.cs`를 열고 클레임 컨트롤러에 `[Authorize]` 태그를 추가합니다. OWIN는 권한 부여 태그를 호출했을 때 실행되도록 구성된 마지막 정책을 선택합니다.

```C#
// Controllers\HomeController.cs

// You can use the Authorize decorator to execute a policy if the user is not already signed in the app.
[Authorize]
public ActionResult Claims()
{
  ...
```

또한 앱에서 사용자를 로그아웃하는 데 OWIN을 사용할 수도 있습니다. `Controllers\AccountController.cs`:

```C#
// Controllers\AccountController.cs

public void SignOut()
{
    // To sign out the user, you should issue an OpenIDConnect sign out request
    if (Request.IsAuthenticated)
    {
        IEnumerable<AuthenticationDescription> authTypes = HttpContext.GetOwinContext().Authentication.GetAuthenticationTypes();
        HttpContext.GetOwinContext().Authentication.SignOut(authTypes.Select(t => t.AuthenticationType).ToArray());
        Request.GetOwinContext().Authentication.GetAuthenticationTypes();
    }
}
```

## 사용자 정보 표시
OpenID Connect를 사용하여 사용자를 인증할 때 Azure AD는 **클레임**을 포함하는 ID 토큰을 앱에 반환합니다. 이는 사용자에 대한 어설션입니다. 클레임을 사용하여 앱 개인 설정을 수행할 수 있습니다.

`Controllers\HomeController.cs` 파일을 엽니다. `ClaimsPrincipal.Current` 보안 주체 개체를 통해 컨트롤러의 사용자 클레임에 액세스할 수 있습니다.

```C#
// Controllers\HomeController.cs

[Authorize]
public ActionResult Claims()
{
    Claim displayName = ClaimsPrincipal.Current.FindFirst(ClaimsPrincipal.Current.Identities.First().NameClaimType);
    ViewBag.DisplayName = displayName != null ? displayName.Value : string.Empty;
    return View();
}
```

동일한 방식으로 응용 프로그램에서 수신하는 클레임을 액세스할 수 있습니다. 앱이 수신하는 모든 클레임 목록은 **클레임** 페이지에서 사용할 수 있습니다.

## 샘플 앱 실행
마지막으로 앱을 빌드하고 실행할 수 있습니다. 메일 주소 또는 사용자 이름을 사용하여 앱에 등록합니다. 로그아웃했다가 동일한 사용자로 다시 로그인합니다. 해당 사용자의 프로필을 편집합니다. 로그아웃했다가 다른 사용자로 등록합니다. **클레임** 탭에 표시되는 정보가 정책에 구성한 정보와 어떻게 일치하는지 확인합니다.

## 소셜 IDP 추가
현재, 앱은 **로컬 계정**을 사용하여 사용자 등록 및 로그인만 지원합니다. 이들은 사용자 이름 및 암호를 사용하는 B2C 디렉터리에 저장된 계정입니다. Azure AD B2C를 사용하면 코드를 변경하지 않고도 다른 **IDP(ID 공급자)**에 대한 지원을 추가할 수 있습니다.

소셜 IDP를 앱에 추가하려면 이 문서 중에서 상세한 지침을 수행하여 시작합니다. 지원하려는 각 IDP의 경우 해당 시스템에서 응용 프로그램을 등록하고 클라이언트 ID를 얻어야 합니다.

* [Facebook을 IDP로 설정](active-directory-b2c-setup-fb-app.md)
* [Google을 IDP로 설정](active-directory-b2c-setup-goog-app.md)
* [Amazon을 IDP로 설정](active-directory-b2c-setup-amzn-app.md)
* [LinkedIn을 IDP로 설정](active-directory-b2c-setup-li-app.md)

B2C 디렉터리에 ID 공급자를 추가한 후 [정책 참조 문서](active-directory-b2c-reference-policies.md)에서 설명한 대로 새 IDP를 포함하도록 세 가지 정책을 각각 편집해야 합니다. 정책을 저장한 후 앱을 다시 실행합니다. ID 환경 각각에서 로그인 및 등록으로 추가된 새 IDP가 표시되어야 합니다.

정책을 실험하고 샘플 앱에서 영향을 확인할 수 있습니다. IDP를 추가 또는 제거하거나 응용 프로그램 클레임을 조작하거나 등록 특성을 변경합니다. 어떻게 정책, 인증 요청 및 OWIN을 모두 함께 연결하는지 확인할 수 있을 때까지 실험해 보세요.

참조를 위해 완성된 샘플(사용자 구성 값 제외)이 [.zip 파일로 제공](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIdConnect-DotNet/archive/complete.zip)됩니다. 또한 GitHub에서 복제할 수 있습니다.

```
git clone --branch complete https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIdConnect-DotNet.git
```

<!--

## Next steps

You can now move on to more advanced B2C topics. You might try:

[Call a web API from a web app]()

[Customize the UX for a B2C app]()

-->

<!---HONumber=AcomDC_0727_2016-->