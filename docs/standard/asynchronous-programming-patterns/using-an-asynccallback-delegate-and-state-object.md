---
title: 使用 AsyncCallback 委托和状态对象
ms.date: 03/30/2017
ms.technology: dotnet-standard
dev_langs:
- csharp
- vb
helpviewer_keywords:
- asynchronous programming, delegates
- AsyncCallback delegate, samples
- asynchronous programming, state objects
- IAsyncResult interface, samples
ms.assetid: e3e5475d-c5e9-43f0-928e-d18df8ca1f1d
author: rpetrusha
ms.author: ronpet
ms.openlocfilehash: 46b7839a6bd0086a8ec82e416cdf7aed05707390
ms.sourcegitcommit: 4b6490b2529707627ad77c3a43fbe64120397175
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/10/2018
ms.locfileid: "44259897"
---
# <a name="using-an-asynccallback-delegate-and-state-object"></a>使用 AsyncCallback 委托和状态对象
使用 <xref:System.AsyncCallback> 委托处理单独线程中的异步操作结果时，可以使用状态对象，在两个回调之间传递信息，并检索最终结果。 本主题通过扩展[使用 AsyncCallback 委托结束异步操作](../../../docs/standard/asynchronous-programming-patterns/using-an-asynccallback-delegate-to-end-an-asynchronous-operation.md)中的示例，展示了此做法。  
  
## <a name="example"></a>示例  
 下面的代码示例展示了如何使用 <xref:System.Net.Dns> 类中的异步方法，检索用户指定计算机的域名系统 (DNS) 信息。 此示例定义并使用 `HostRequest` 类存储状态信息。 `HostRequest` 对象是针对用户输入的每个计算机名进行创建。 此对象传递给 <xref:System.Net.Dns.BeginGetHostByName%2A> 方法。 每当请求完成时，都会调用 `ProcessDnsInformation` 方法。 `HostRequest` 对象是使用 <xref:System.IAsyncResult.AsyncState%2A> 属性进行检索。 `ProcessDnsInformation` 方法使用 `HostRequest` 对象，存储请求返回的 <xref:System.Net.IPHostEntry> 或请求抛出的 <xref:System.Net.Sockets.SocketException>。 所有请求完成后，应用会循环访问 `HostRequest`对象，并显示 DNS 信息或 <xref:System.Net.Sockets.SocketException> 错误消息。  
  
 [!code-csharp[AsyncDesignPattern#5](../../../samples/snippets/csharp/VS_Snippets_CLR/AsyncDesignPattern/CS/AsyncDelegateWithStateObject.cs#5)]
 [!code-vb[AsyncDesignPattern#5](../../../samples/snippets/visualbasic/VS_Snippets_CLR/AsyncDesignPattern/VB/AsyncDelegateWithStateObject.vb#5)]  
  
## <a name="see-also"></a>请参阅

- [基于事件的异步模式 (EAP)](../../../docs/standard/asynchronous-programming-patterns/event-based-asynchronous-pattern-eap.md)  
- [基于事件的异步模式概述](../../../docs/standard/asynchronous-programming-patterns/event-based-asynchronous-pattern-overview.md)  
- [使用 AsyncCallback 委托停止异步操作](../../../docs/standard/asynchronous-programming-patterns/using-an-asynccallback-delegate-to-end-an-asynchronous-operation.md)
