# Azure 클라우드 서비스에 대한 사용자 지정 도메인 이름 구성
> [!NOTE]
> 새로운 Azure의 [안내 방식 연습](http://support.microsoft.com/kb/2990804)을 사용하면 작업을 보다 빠르게 수행할 수 있습니다.  이 연습을 통해 사용자 지정 도메인 이름을 연결하고 SSL을 사용하여 Azure 클라우드 서비스 또는 Azure 웹 사이트와의 통신을 보호하는 등의 작업을 매우 쉽게 완료할 수 있습니다.
> 
> 

Azure에서 응용 프로그램을 만들면 Azure에서는 사용자가 http://&lt;*myapp*>.cloudapp.net 등의 URL에서 응용 프로그램에 액세스할 수 있도록 cloudapp.net 도메인에 하위 도메인을 제공합니다. 그러나 contoso.com 등의 고유한 도메인 이름에도 응용 프로그램을 표시할 수 있습니다.

> [!NOTE]
> 이 작업의 절차는 Azure 클라우드 서비스에 적용됩니다. 저장소 계정의 경우 [Azure 저장소 계정에 대한 사용자 지정 도메인 이름 구성]을 참조하세요(../articles/storage-custom-domain-name.md). 웹 사이트의 경우 [Azure 웹 사이트에 대한 사용자 지정 도메인 이름 구성](../articles/app-service-web/web-sites-custom-domain-name.md).
> 
> 

## CNAME 및 A 레코드 이해
CNAME(또는 별칭 레코드) 및 A 레코드는 둘 다 도메인 이름을 특정 서버(또는 이 경우 서비스)에 연결할 수 있게 해 주지만 다르게 작동합니다. Azure 클라우드 서비스와 함께 A 레코드를 사용하는 경우의 몇 가지 특정 고려 사항도 있으며, 사용할 레코드를 결정하기 전에 고려해야 합니다.

### CNAME 또는 별칭 레코드
CNAME 레코드는 *specific* 도메인(예: **contoso.com** 또는 **www.contoso.com**)을 정식 도메인 이름에 매핑합니다. 이 경우 정식 도메인 이름은 Azure 호스티드 응용 프로그램의 **&lt;myapp>.cloudapp.net** 도메인 이름입니다. CNAME을 만들면 **&lt;myapp>.cloudapp.net**에 대한 별칭이 만들어집니다. CNAME 항목은 **&lt;myapp>.cloudapp.net** 서비스의 IP 주소로 자동으로 확인되므로 클라우드 서비스의 IP 주소가 변경될 경우 특별한 조치를 수행할 필요가 없습니다.

> [!NOTE]
> contoso.com과 같은 루트 이름이 아닌 www.contoso.com과 같은 CNAME 레코드를 사용할 경우 일부 도메인 등록 기관에서만 하위 도메인을 매핑할 수 있습니다. CNAME 레코드에 대한 자세한 내용은 등록 기관에서 제공하는 설명서인 <a href="http://en.wikipedia.org/wiki/CNAME_record">CNAME 레코드에 대한 Wikipedia 항목</a>(영문) 또는 <a href="http://tools.ietf.org/html/rfc1035">IETF 도메인 이름 - 구현 및 사양</a>(영문) 문서를 참조하세요.
> 
> 

### A 레코드
A 레코드는 **contoso.com**, **www.contoso.com** 등의 도메인이나 **.contoso.com** 등의 *or a wildcard domain*을 IP 주소에 매핑합니다. Azure 클라우드 서비스의 경우 서비스의 가상 IP입니다. 따라서 A 레코드가 CNAME 레코드보다 나은 주요 장점은 **.contoso.com**과 같이 와일드카드를 사용하는 항목을 사용할 수 있다는 것입니다. 이러한 항목은 **mail.contoso.com**, **login.contoso.com**, **www.contso.com** 등의 여러 하위 도메인에 대한 요청을 처리합니다.

> [!NOTE]
> A 레코드는 고정 IP 주소에 매핑되므로 변경 내용을 클라우드 서비스의 IP 주소로 자동으로 확인할 수 없습니다. 빈 슬롯(프로덕션 또는 스테이징)에 처음 배포할 때 클라우드 서비스에서 사용되는 IP 주소가 할당됩니다. 슬롯에 대한 배포를 삭제하면 Azure에서 IP 주소를 해제하며, 나중에 슬롯에 배포할 때 새 IP 주소가 지정될 수 있습니다.
> 
> 편의상, 스테이징 배포와 프로덕션 배포 간에 전환하거나 기존 배포의 바로 업그레이드를 수행하는 경우 주어진 배포 슬롯(프로덕션 또는 스테이징)의 IP 주소가 지속됩니다. 이러한 작업을 수행하는 방법에 대한 자세한 내용은 [클라우드 서비스를 관리하는 방법]을 참조하세요.(../articles/cloud-services-how-to-manage.md).
> 
> 

## 사용자 지정 도메인에 대한 CNAME 레코드 추가
CNAME 레코드를 만들려면 등록 기관에서 제공한 도구를 사용하여 사용자 지정 도메인에 대한 새 항목을 DNS 테이블에 추가해야 합니다. 각 등록 기관은 서로 유사하지만 약간 다른 방법으로 CNAME 레코드를 지정하지만 개념은 동일합니다.

1. 이러한 방법 중 하나를 사용하여 클라우드 서비스에 할당된 **.cloudapp.net** 도메인 이름을 찾습니다.
   
   * [Azure 관리 포털]에 로그인하고 클라우드 서비스를 선택한 다음 **대시보드**를 선택하고 **빠른 보기** 섹션에서 **사이트 URL** 항목을 찾습니다.
     
            ![사이트 URL을 표시하는 빠른 보기 섹션][csurl]
   * [Azure Powershell]을 설치 및 구성한(../articles/install-configure-powershell.md)후 다음 명령을 사용합니다.
     
     Get-AzureDeployment -ServiceName yourservicename | Select Url
   
   CNAME 레코드를 만들 때 필요하므로 두 방법 중 하나에서 반환된 URL에 사용된 도메인 이름을 저장합니다.
2. DNS 등록 기관의 웹 사이트에 로그온한 다음 DNS 관리 페이지로 이동합니다. **도메인 이름**, **DNS** 또는 **이름 서버 관리**로 레이블이 지정된 사이트의 링크 또는 영역을 찾습니다.
3. 이제 CNAME을 선택하거나 입력할 수 있는 위치를 찾습니다. 드롭다운에서 레코드 유형을 선택하거나 고급 설정 페이지로 이동해야 할 수도 있습니다. **CNAME**, **별칭** 또는 **하위 도메인**과 같은 단어를 찾아야 합니다.
4. **www.customdomain.com**에 대한 별칭을 만들려는 경우 **www**와 같은 CNAME에 대한 도메인 또는 하위 도메인 별칭도 제공해야 합니다. 루트 도메인에 대한 별칭을 만들려는 경우 등록 기관의 DNS 도구에서 '**@**' 기호로 표시될 수도 있습니다.
5. 정식 호스트 이름을 제공해야 합니다. 이 경우에는 응용 프로그램의 **cloudapp.net** 도메인입니다.

예를 들어 다음 CNAME 레코드는 **www.contoso.com**의 모든 트래픽을 배포된 응용 프로그램의 사용자 지정 도메인 이름인 **contoso.cloudapp.net**으로 전달합니다.

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tr>
<td><strong>별칭/호스트 이름/하위 도메인</strong></td>
<td><strong>정식 도메인</strong></td>
</tr>
<tr>
<td>www</td>
<td>contoso.cloudapp.net</td>
</tr>
</table>

**www.contoso.com** 방문자에게는 실제
호스트(contoso.cloudapp.net)가 표시되지 않으므로 전달 프로세스가 최종 사용자에게
표시되지 않습니다.

> [!NOTE]
> 위의 예제는 <strong>www</strong> 하위 도메인의 트래픽에만 적용됩니다. CNAME 레코드와 함께 와일드카드를 사용할 수 없으므로 각 도메인/하위 도메인에 대해 하나의 CNAME을 만들어야 합니다. *.contoso.com 등의 하위 도메인에서 cloudapp.net 주소로 트래픽을 보내려는 경우 DNS 설정에서 <strong>URL 리디렉션</strong> 또는 <strong>URL 전달</strong> 항목을 구성하거나 A 레코드를 만들 수 있습니다.
> 
> 

## 사용자 지정 도메인에 대한 A 레코드 추가
A 레코드를 만들려면 먼저 클라우드 서비스의 가상 IP 주소를 찾아야 합니다. 그런 다음 등록 기관에서 제공한 도구를 사용하여 사용자 지정 도메인에 대한 새 항목을 DNS 테이블에 추가합니다. 각 등록 기관은 서로 유사하지만 약간 다른 방법으로 A 레코드를 지정하지만 개념은 동일합니다.

1. 다음 방법 중 하나를 사용하여 클라우드 서비스의 IP 주소를 가져옵니다.
   
   * [Azure 관리 포털]에 로그인하고 클라우드 서비스를 선택한 다음 **대시보드**를 선택하고 **빠른 보기** 섹션에서 **공용 VIP(가상 IP) 주소** 항목을 찾습니다.
     
            ![VIP를 표시하는 빠른 보기 섹션][vip]
   * [Azure Powershell]을 설치 및 구성한(../articles/install-configure-powershell.md)후 다음 명령을 사용합니다.
     
      get-azurevm -servicename yourservicename | get-azureendpoint -VM {$_.VM} | select Vip
     
     여러 끝점이 클라우드 서비스에 연결되어 있는 경우 IP 주소가 포함된 여러 줄을 받지만 모두 동일한 주소가 표시되어야 합니다.
   
   A 레코드를 만들 때 필요하므로 IP 주소를 저장합니다.
2. DNS 등록 기관의 웹 사이트에 로그온한 다음 DNS 관리 페이지로 이동합니다. **도메인 이름**, **DNS** 또는 **이름 서버 관리**로 레이블이 지정된 사이트의 링크 또는 영역을 찾습니다.
3. 이제 A 레코드를 선택하거나 입력할 수 있는 위치를 찾습니다. 드롭다운에서 레코드 유형을 선택하거나 고급 설정 페이지로 이동해야 할 수도 있습니다.
4. 이 A 레코드를 사용할 도메인 또는 하위 도메인을 선택하거나 입력합니다. 예를 들어 **www.customdomain.com**에 대한 별칭을 만들려는 경우 **www**를 선택합니다. 모든 하위 도메인에 대한 와일드카드 항목을 만들려면 '*****'를 입력합니다. 그러면 **mail.customdomain.com**, **login.customdomain.com**, **www.customdomain.com** 등의 모든 하위 도메인이 포함됩니다.
   
   루트 도메인에 대한 A 레코드를 만들려는 경우 등록 기관의 DNS 도구에서 '**@**' 기호로 표시될 수도 있습니다.
5. 제공된 필드에 클라우드 서비스의 IP 주소를 입력합니다. 그러면 A 레코드에 사용된 도메인 항목이 클라우드 서비스 배포의 IP 주소와 연결됩니다.

예를 들어 다음 A 레코드는 모든 트래픽을 **contoso.com**에서 배포된 응용 프로그램의 IP 주소인 **137.135.70.239**로 전달합니다.

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tr>
<td><strong>호스트 이름/하위 도메인</strong></td>
<td><strong>IP 주소</strong></td>
</tr>
<tr>
<td>@</td>
<td>137.135.70.239</td>
</tr>
</table>

이 예제에서는 루트 도메인에 대한 A 레코드를 만드는 방법을 보여 줍니다. 모든 하위 도메인을 포함할 와일드카드 항목을 만들려면 '*****'를 하위 도메인으로 입력합니다.

## 다음 단계
* [클라우드 서비스를 관리하는 방법](../articles/cloud-services/cloud-services-how-to-manage.md)
* [CDN 콘텐츠를 사용자 지정 도메인에 매핑하는 방법][CDN 콘텐츠를 사용자 지정 도메인에 매핑하는 방법]

[사용자 지정 도메인에 응용 프로그램 표시]: #access-app
[사용자 지정 도메인에 대한 CNAME 레코드 추가]: #add-cname
[사용자 지정 도메인에 데이터 표시]: #access-data
[VIP 교환]: http://msdn.microsoft.com/library/ee517253.aspx
[하위 도메인을 저장소 계정과 연결하는 CNAME 레코드 만들기]: #create-cname
[Azure 관리 포털]: https://manage.windowsazure.com
[사용자 지정 도메인 유효성 검사 대화 상자]: http://i.msdn.microsoft.com/dynimg/IC544437.jpg
[CDN 콘텐츠를 사용자 지정 도메인에 매핑하는 방법]: http://msdn.microsoft.com/library/windowsazure/gg680307.aspx
[vip]: ./media/custom-dns/csvip.png
[csurl]: ./media/custom-dns/csurl.png

<!--HONumber=49-->