---
title: Windows Communication Foundation 使用展望：使未来迁移轻而易举
ms.date: 03/30/2017
ms.assetid: f49664d9-e9e0-425c-a259-93f0a569d01b
ms.openlocfilehash: 171a31b375eae4c032849c2a1c2090f5d9ff856f
ms.sourcegitcommit: 586dbdcaef9767642436b1e4efbe88fb15473d6f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2018
ms.locfileid: "48837364"
---
# <a name="anticipating-adopting-the-windows-communication-foundation-easing-future-migration"></a>Windows Communication Foundation 使用展望：使未来迁移轻而易举
若要确保新的 ASP.NET 应用程序到 WCF 为简化未来迁移，请遵循之前提出的建议，以及以下建议。  
  
## <a name="protocols"></a>协议  
 禁用 ASP.NET 2.0 对 SOAP 1.2 的支持：  
  
```xml  
<configuration>  
     <system.web>  
      <webServices >  
          <protocols>  
           <remove name="HttpSoap12"/>  
          </protocols>    
      </webServices>  
     </system.web>   
</configuration>  
```  
  
 执行此操作是建议的因为 WCF 要求符合不同的协议，如 SOAP 1.1 和 SOAP 1.2，通过使用不同的终结点的消息。 如果 ASP.NET 2.0 Web 服务配置为支持 SOAP 1.1 和 SOAP 1.2，这是默认配置中，则不能对其进行迁移到单个 WCF 终结点是肯定的原始地址的正向是与所有 ASP.NET Web 兼容服务的现有客户端。 另外，选择 SOAP 1.2 而非 SOAP 1.1 将更加严格地限制服务的客户。  
  
## <a name="service-development"></a>服务开发  
 WCF 允许你通过应用定义服务协定<xref:System.ServiceModel.ServiceContractAttribute>到接口或类。 建议将此属性应用于接口而不是类，因为这样可以为可由任意数量的类以不同方式实现的协定创建一个定义。 ASP.NET 2.0 支持将 <xref:System.Web.Services.WebService> 属性应用到接口和类的选项。 但是，如前所述，ASP.NET 2.0 中存在缺陷，如果将 <xref:System.Web.Services.WebService> 属性应用到接口而不是类，则此属性的 Namespace 参数将不起任何作用。 由于它是通常建议修改的默认值，从服务命名空间`http://tempuri.org`，使用的 Namespace 参数<xref:System.Web.Services.WebService>属性，其中一个应继续定义 ASP.NET Web 服务通过应用<xref:System.ServiceModel.ServiceContractAttribute>到接口或类的属性。  
  
-   在定义那些接口的方法中尽量少使用代码。 允许它们将其工作委托给其他类。 新的 WCF 服务类型然后也可以将委托到这些类其大量的工作。  
  
-   使用 `MessageName` 的 <xref:System.Web.Services.WebMethodAttribute> 参数为服务的操作提供显式名称。  
  
    ```  
    [WebMethod(MessageName="ExplicitName")]  
    string Echo(string input);  
    ```  
  
     执行此操作很重要，因为 ASP.NET 中的操作的默认名称是由 WCF 提供的默认名称不同。 通过提供显式名称，可以避免依赖默认名称。  
  
-   不要使用多态方法实现 ASP.NET Web 服务操作，因为 WCF 不支持使用多态方法实现操作。  
  
-   使用 <xref:System.Web.Services.Protocols.SoapDocumentMethodAttribute> 为 SOAPAction HTTP 标头提供显式值，HTTP 请求将通过此标头路由到方法。  
  
    ```  
    [WebMethod]  
    [SoapDocumentMethod(RequestElementName="ExplicitAction")]  
    string Echo(string input);  
    ```  
  
     采用这种方法可摆脱依赖于默认使用的 ASP.NET 和 WCF 相同的 SOAPAction 值。  
  
-   避免使用 SOAP 扩展。 如果 SOAP 扩展是必需的则确定是否正在为其考虑它们的用途是一项功能已提供的 WCF。 如果这确实是这种情况，然后重新考虑该选择不立即采用 WCF。  
  
## <a name="state-management"></a>状态管理  
 避免必须在服务中保持状态。 不仅保持状态会破坏应用程序的可伸缩性，而且 ASP.NET 和 WCF 的状态管理机制会大不相同，尽管 WCF 在 ASP.NET 兼容模式下支持 ASP.NET 机制。  
  
## <a name="exception-handling"></a>异常处理  
 在设计服务发送和接收的数据类型的结构时，请同时设计表示异常的不同类型的结构，这些异常可能发生在人们希望传送到客户端的服务中。  
  
```  
[Serializable]  
[XmlRoot(  
     Namespace="ExplicitNamespace", IsNullable=true)]  
    public partial class AnticipatedException {  
  
     private string anticipatedExceptionInformationField;  
  
     public string AnticipatedExceptionInformation {  
      get {  
          return this.anticipatedExceptionInformationField;  
          }  
      set {  
          this.anticipatedExceptionInformationField = value;  
          }  
     }  
}  
```  
  
 使这些类能够将其自身序列化为 XML：  
  
```  
public XmlNode ToXML()  
{  
     XmlSerializer serializer = new XmlSerializer(  
      typeof(AnticipatedException));  
     MemoryStream memoryStream = new MemoryStream();  
     XmlTextWriter writer = new XmlTextWriter(  
     memoryStream, UnicodeEncoding.UTF8);  
     serializer.Serialize(writer, this);  
     XmlDocument document = new XmlDocument();  
     document.LoadXml(new string(  
     UnicodeEncoding.UTF8.GetChars(  
     memoryStream.GetBuffer())).Trim());  
    return document.DocumentElement;  
}  
```  
  
 然后，可使用这些类为显式引发的 <xref:System.Web.Services.Protocols.SoapException> 实例提供详细信息：  
  
```  
AnctipatedException exception = new AnticipatedException();  
exception.AnticipatedExceptionInformation = "…";  
throw new SoapException(  
     "Fault occurred",  
     SoapException.ClientFaultCode,  
     Context.Request.Url.AbsoluteUri,  
     exception.ToXML());  
```  
  
 可通过 WCF 随时重用这些异常类<xref:System.ServiceModel.FaultException%601>类来引发新 `FaultException<AnticipatedException>(anticipatedException);`  
  
## <a name="security"></a>安全性  
 下面是一些安全建议。  
  
-   避免使用 ASP.NET 2.0 配置文件，作为使用它们将限制使用的 ASP.NET 集成模式下，如果将服务迁移到 WCF。  
  
-   避免使用 Acl 来控制访问的服务作为 ASP.NET Web 服务支持使用 Internet Information Services (IIS) 的 Acl，WCF 不会，因为 ASP.NET Web 服务依赖 IIS 进行承载，并且 WCF 不一定需要在 IIS 中承载。  
  
-   请务必考虑使用 ASP.NET 2.0 角色提供程序来授权对服务资源的访问。  
  
## <a name="see-also"></a>请参阅  
 [预期采用 Windows Communication Foundation：便于以后集成](../../../../docs/framework/wcf/feature-details/anticipating-adopting-the-wcf-easing-future-integration.md)
