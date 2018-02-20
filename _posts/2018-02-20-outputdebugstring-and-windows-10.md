---
ID: 461
post_title: >
  Win32 library on Windows 10 Creators
  edition using Outputdebugstring
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2018/02/outputdebugstring-and-windows-10/
published: true
post_date: 2018-02-20 22:08:20
---
> At a readers request, I wrote this post describing a 'Windows 10 Creators Update' quirk/feature.

## the symptom:

While testing [Debugview++][1] on Windows Windows 10 1703 Creators Update Build 15063.413 (the latest version at the time) one of my users found that the application hangs at startup for about 20 seconds, before actually continuing the startup. After this it was working normally.

The bug was reported [here][2]. On the version before that Windows 10 version 1607 14393.1358 this problem did not occur. Now [Debugview++][1] is kind of special in that it listens to a 'debugger' API, namely to receive the '[OutputDebugString][3]' messages. For those of you not familiar with this API, it is win32 facility to output debug message (as the name suggests).

Normally debugview++ receives these messages from another process. The mechanism to deliver the message is quite [primitive][4], it is a fixed 4096 bytes buffer and the caller of OutputDebugString can briefly be blocked if messages are not immediately read by the debug application.

Now, I've optimized debugview++ to read message as fast as possible and to unblock the traced application asap. I wondered what could be causing debugview++ **itself** to block for 20 seconds? The 20 seconds were curious in itself because the OutputDebugString API has a 10-second timeout, so it looked awfully like two message-timeouts; could I have made a mistake and was I accidentally using the OutputDebugString myself within debugview++ ? Of course that would be problematic, as that could potentially cause an infinite recursion problem.

After some investigation this turned out, *debugview++ itselt* was not sending any messages, at least not from its code, but the win32 libraries (specifically oleaut32.dll) that where loaded implicitly, can now call into this OutputdebugString API.

So the problem was introduced because in Windows 10 1703 the Win32 API itself started using OutputDebugString internally. Below is a stackdump from the process while it was hanging (created using taskmanager's 'Create dump file' feature).

<img src="http://nullptr.nl/wp-content/uploads/2018/02/create_dump.png" alt="" width="626" height="674" class="aligntop size-full wp-image-1022" />

<div>
</div>

        ntdll.dll!_NtWaitForSingleObject@12�()  Unknown
        KERNELBASE.dll!WaitForSingleObjectEx()  Unknown
        KERNELBASE.dll!OutputDebugStringA() Unknown
    >   KERNELBASE.dll!OutputDebugStringW() Unknown
        oleaut32.dll!wil::details::LogFailure(void *,unsigned int,char const *,char const *,char const *,void *,enum wil::FailureType,long,unsigned short const *,bool,unsigned short *,unsigned int,char *,unsigned int,struct wil::FailureInfo *) Unknown
        oleaut32.dll!wil::details::ReportFailure(void *,unsigned int,char const *,char const *,char const *,void *,enum wil::FailureType,long,unsigned short const *,enum wil::details::ReportFailureOptions)   Unknown
        oleaut32.dll!wil::details::ReportFailure_Hr(void *,unsigned int,char const *,char const *,char const *,void *,enum wil::FailureType,long)   Unknown
        oleaut32.dll!wil::details::in1diag3::Return_Hr_NoOriginate(void *,unsigned int,char const *,long)   Unknown
        oleaut32.dll!GetTypeInfoOfIIDFwd(struct _GUID const &,struct ITypeInfo * *,int) Unknown
        oleaut32.dll!CProxyWrapper::Connect(struct IRpcChannelBuffer *) Unknown
        combase.dll!CStdMarshal::ConnectProxyToChannel(tagIPIDEntry * pEntry, OXIDEntry * pOXIDEntry, const _GUID & ipid) Line 3206 C++
        [Inline Frame] combase.dll!CStdMarshal::ConnectCliIPIDEntry(tagSTDOBJREF *) Line 3055   C++
        combase.dll!CStdMarshal::MakeCliIPIDEntry(const _GUID & riid, tagSTDOBJREF * pStd, OXIDEntry * pOXIDEntry, tagIPIDEntry * * ppEntry) Line 2812  C++
        combase.dll!CStdMarshal::UnmarshalIPID(const _GUID & riid, tagSTDOBJREF * pStd, OXIDEntry * pOXIDEntry, void * * ppv) Line 2340 C++
        combase.dll!CStdMarshal::UnmarshalObjRef(tagOBJREF & objref, void * * ppv) Line 2208    C++
        combase.dll!UnmarshalSwitch(void * pv) Line 1843    C++
        combase.dll!UnmarshalObjRef(tagOBJREF & objref, EffectiveUnmarshalingPolicy policy, void * * ppv, int fBypassActLock, CStdMarshal * * ppStdMarshal) Line 1985   C++
        combase.dll!_CoUnmarshalInterface(IStream * pStm, bool bForceStrongPolicy, const _GUID & riid, void * * ppv) Line 1730  C++
        combase.dll!CoUnmarshalInterface(IStream * pStm, const _GUID & riid, void * * ppv) Line 1773    C++
        combase.dll!ActivationPropertiesOut::OutSerializer::UnmarshalAtIndex(unsigned long index, bool bIsWinRTOutofproc) Line 3139 C++
        combase.dll!ActivationPropertiesOut::GetObjectInterfaces(unsigned long cIfs, unsigned long actvflags, tagMULTI_QI * pMultiQi) Line 2746 C++
        combase.dll!ICoCreateInstanceEx(const _GUID & OriginalClsid, IUnknown * punkOuter, unsigned long dwClsCtx, _COSERVERINFO * pServerInfo, unsigned long dwCount, unsigned long dwActvFlags, tagMULTI_QI * pResults, ActivationPropertiesIn * pActIn) Line 1971    C++
        combase.dll!CComActivator::DoCreateInstance(const _GUID & Clsid, IUnknown * punkOuter, unsigned long dwClsCtx, _COSERVERINFO * pServerInfo, unsigned long dwCount, tagMULTI_QI * pResults, ActivationPropertiesIn * pActIn) Line 384    C++
        [Inline Frame] combase.dll!CoCreateInstanceEx(const _GUID &) Line 176   C++
        combase.dll!CoCreateInstance(const _GUID & rclsid, IUnknown * pUnkOuter, unsigned long dwContext, const _GUID & riid, void * * ppv) Line 120    C++
        uiautomationcore.dll!BlockingCoreDispatcher::CreateAndRegisterBlockingCore(void)    Unknown
        uiautomationcore.dll!BlockingCoreDispatcher::FinalConstruct(void)   Unknown
        uiautomationcore.dll!ATL::CComCreator<class ATL::CComObject<class CUIAutomation8> >::CreateInstance(void *,struct _GUID const &,void * *)   Unknown
        uiautomationcore.dll!ATL::CComCreator2<class ATL::CComCreator<class ATL::CComObject<class CUIAutomation8> >,class ATL::CComFailCreator<-2147221232> >::CreateInstance(void *,struct _GUID const &,void * *) Unknown
        uiautomationcore.dll!ATL::CComClassFactory::CreateInstance(struct IUnknown *,struct _GUID const &,void * *) Unknown
        combase.dll!CServerContextActivator::CreateInstance(IUnknown * pUnkOuter, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties) Line 872 C++
        combase.dll!ActivationPropertiesIn::DelegateCreateInstance(IUnknown * pUnkOuter, IActivationPropertiesOut * * ppActPropsOut) Line 1931  C++
        combase.dll!CApartmentActivator::CreateInstance(IUnknown * pUnkOuter, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties) Line 2163    C++
        combase.dll!CProcessActivator::CCICallback(unsigned long dwContext, IUnknown * pUnkOuter, ActivationPropertiesIn * pActIn, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties) Line 1631   C++
        combase.dll!CProcessActivator::AttemptActivation(ActivationPropertiesIn * pActIn, IUnknown * pUnkOuter, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties, HRESULT(__stdcallCProcessActivator::*)(unsigned long, IUnknown *, ActivationPropertiesIn *, IActivationPropertiesIn *, IActivationPropertiesOut * *) pfnCtxActCallback, unsigned long dwContext) Line 1518 C++
        combase.dll!CProcessActivator::ActivateByContext(ActivationPropertiesIn * pActIn, IUnknown * pUnkOuter, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties, HRESULT(__stdcallCProcessActivator::*)(unsigned long, IUnknown *, ActivationPropertiesIn *, IActivationPropertiesIn *, IActivationPropertiesOut * *) pfnCtxActCallback) Line 1384  C++
        combase.dll!CProcessActivator::CreateInstance(IUnknown * pUnkOuter, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties) Line 1262  C++
        combase.dll!ActivationPropertiesIn::DelegateCreateInstance(IUnknown * pUnkOuter, IActivationPropertiesOut * * ppActPropsOut) Line 1931  C++
        combase.dll!CClientContextActivator::CreateInstance(IUnknown * pUnkOuter, IActivationPropertiesIn * pInActProperties, IActivationPropertiesOut * * ppOutActProperties) Line 561 C++
        combase.dll!ActivationPropertiesIn::DelegateCreateInstance(IUnknown * pUnkOuter, IActivationPropertiesOut * * ppActPropsOut) Line 1983  C++
        combase.dll!ICoCreateInstanceEx(const _GUID & OriginalClsid, IUnknown * punkOuter, unsigned long dwClsCtx, _COSERVERINFO * pServerInfo, unsigned long dwCount, unsigned long dwActvFlags, tagMULTI_QI * pResults, ActivationPropertiesIn * pActIn) Line 1915    C++
        combase.dll!CComActivator::DoCreateInstance(const _GUID & Clsid, IUnknown * punkOuter, unsigned long dwClsCtx, _COSERVERINFO * pServerInfo, unsigned long dwCount, tagMULTI_QI * pResults, ActivationPropertiesIn * pActIn) Line 384    C++
        [Inline Frame] combase.dll!CoCreateInstanceEx(const _GUID &) Line 176   C++
        combase.dll!CoCreateInstance(const _GUID & rclsid, IUnknown * pUnkOuter, unsigned long dwContext, const _GUID & riid, void * * ppv) Line 120    C++
        tiptsf.dll!CImmersiveFocusTracker::_HandleAutomationEvent(unsigned long,struct HWND__ *,long)   Unknown
        tiptsf.dll!CImmersiveFocusTracker::HandleAutomationEvent(unsigned long,struct HWND__ *,long)    Unknown
        tiptsf.dll!TabletMsgWndProc(int,unsigned int,long)  Unknown
        user32.dll!_DispatchHookW@16�() Unknown
        user32.dll!_CallHookWithSEH@16�()   Unknown
        user32.dll!___fnHkINLPMSG@4�()  Unknown
        ntdll.dll!_KiUserCallbackDispatcher@12�()   Unknown
        DebugView++.exe!WTL::CCommandBarCtrlImpl<WTL::CCommandBarCtrl,WTL::CCommandBarCtrlBase,ATL::CWinTraits<1442840576,0> >::MessageHookProc(int nCode, unsigned int wParam, long lParam) Line 2747  C++
        user32.dll!_DispatchHookW@16�() Unknown
        user32.dll!_CallHookWithSEH@16�()   Unknown
        user32.dll!___fnHkINLPMSG@4�()  Unknown
        ntdll.dll!_KiUserCallbackDispatcher@12�()   Unknown
        DebugView++.exe!WTL::CMessageLoop::Run() Line 1292  C++
        [Inline Frame] DebugView++.exe!fusion::debugviewpp::MessageLoop::Run() Line 61  C++
        DebugView++.exe!fusion::debugviewpp::Main(HINSTANCE__ * cmdShow, HINSTANCE__ *) Line 156    C++
        DebugView++.exe!wWinMain(HINSTANCE__ * hInstance, HINSTANCE__ * hPrevInstance, wchar_t * lpstrCmdLine, int nCmdShow) Line 165   C++
        [Inline Frame] DebugView++.exe!invoke_main() Line 118   C++
        DebugView++.exe!__scrt_common_main_seh() Line 283   C++
        kernel32.dll!75848744() Unknown
        [Frames below may be incorrect and/or missing, no symbols loaded for kernel32.dll]  
        ntdll.dll!__RtlUserThreadStart()    Unknown
        ntdll.dll!__RtlUserThreadStart@8�() Unknown
    

> References:

<https://blogs.msdn.microsoft.com/reiley/2011/07/29/a-debugging-approach-to-outputdebugstring/>

 [1]: https://github.com/CobaltFusion/DebugViewPP
 [2]: https://github.com/CobaltFusion/DebugViewPP/issues/277
 [3]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa363362%28v=vs.85%29.aspx
 [4]: http://www.unixwiz.net/techtips/outputdebugstring.html