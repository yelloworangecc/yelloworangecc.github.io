---
layout: post
title:  "DirectShow Sample Code"
summary: 学习DirectShow时从官网摘录的示例代码,给出了个人注释
featured-img: sleek
---

# Directshow Sample Code

## Play Video File

    HRESULT hr = CoInitialize(NULL); //初始化COM库
    IGraphBuilder* pGraph; //筛选器图生成接口
    HRESULT hr = CoCreateInstance(CLSID_FilterGraph, NULL, 
    CLSCTX_INPROC_SERVER, IID_IGraphBuilder, (void**)&pGraph); //创建Filter Graph Manager
    IMediaControl* pControl;//流控制
    IMediaEvent* pEvent;//获取事件
    hr = pGraph->QueryInterface(IID_IMediaControl, (void**)&pControl);
    hr = pGraph->QueryInterface(IID_IMediaEvent, (void**)&pEvent);
    hr = pGraph->RenderFile(L"C:\\Example.avi", NULL);//生成筛选器图
    hr = pControl->Run();//运行
    long evCode = 0;
    pEvent->WaitForCompletion(INFINITE, &evCode);//等待播放完成
    //释放
    pControl->Release();
    pEvent->Release();
    pGraph->Release();
    CoUninitialize();

## Media Event

    #define WM_GRAPHNOTIFY  WM_APP + 1 //定义专用消息
    IMediaEventEx* g_pEvent = NULL;
    g_pGraph->QueryInterface(IID_IMediaEventEx, (void **)&g_pEvent);//获取接口
    g_pEvent->SetNotifyWindow((OAHWND)g_hwnd, WM_GRAPHNOTIFY, 0);//指定消息接收
    
    //消息分发
    case WM_GRAPHNOTIFY:
        HandleGraphEvent();
        break;
    
    //处理事件
    long evCode;
    LONG_PTR param1, param2;
    HRESULT hr;
    while (SUCCEEDED(g_pEvent->GetEvent(&evCode, &param1, &param2, 0)))
    {
        g_pEvent->FreeEventParams(evCode, param1, param2);
        switch (evCode)
        {
    	case EC_COMPLETE:
    	case EC_USERABORT:
    	case EC_ERRORABORT:
    	    CleanUp();
    	    PostQuitMessage(0);
    	    return;
        }
    }
    
    //释放
    g_pEvent->SetNotifyWindow(NULL, 0, 0);
    g_pEvent->Release();
    g_pEvent = NULL;

## Media Type

    HRESULT CheckMediaType(AM_MEDIA_TYPE *pmt)
    {
        if (pmt == NULL) return E_POINTER;
        // major type
        if (pmt->majortype != MEDIATYPE_Video)
        {
    	return VFW_E_INVALIDMEDIATYPE;
        }
        // subtype
        if (pmt->subtype != MEDIASUBTYPE_RGB24)
        {
    	return VFW_E_INVALIDMEDIATYPE;
        }
        // format type
        if ((pmt->formattype==FORMAT_VideoInfo)&&(pmt->cbFormat>=sizeof(VIDEOINFOHEADER)&&(pmt->pbFormat!=NULL))
        {
    	// Now it's safe to coerce the format block pointer to the
    	// correct structure, as defined by the formattype GUID.
    	VIDEOINFOHEADER* pVIH = (VIDEOINFOHEADER*)pmt->pbFormat;
    	// Examine pVIH (not shown). If it looks OK, return S_OK.
    	return S_OK;
        }
        return VFW_E_INVALIDMEDIATYPE;
    }

## Filter Graph Manager

    //创建Filter Graph Manger实例
    IGraphBuilder* pIGB;
    HRESULT hr = CoCreateInstance(CLSID_FilterGraph, NULL, CLSCTX_INPROC_SERVER, IID_IGraphBuilder, (void **)&pIGB);
    
    //Graph build functions
    IFilterGraph::ConnectDirect //直接链接两个PIN
    IFilterGraph::Connect //链接两个PIN,如果不能直接链接,则使用中间Filter链接.
    IGraphBuilder::Render //从输出PIN开始,生成图像(需要添加必要的Filter)
    IGraphBuilder::RenderFile //从文件播放图像(完整的graph)
    IFilterGraph::AddFilter //添加筛选器

## Smart Connector

    IGraphBuilder::AddSourceFilter
    IGraphBuilder::Render
    IGraphBuilder::RenderFile
    IGraphBuilder::Connect


## Flow

    IMemInputPin
    IAsyncReader


## Media Sample & Memory Allocator

    //访问内存缓冲区方法
    IMediaSample::GetPointer
    IMediaSample::GetSize
    IMediaSample::GetActualDataLength
    //获取Media Sample,上游数据推送
    IMemInputPin::Receive
    //获取指向Media Sample的指针
    IMemAllocator：： GetBuffer


## Filter Status

    IMediaControl::Run
    IMediaControl::Pause
    IMediaControl::Stop



## Request

    IAsyncReader::Request //下游数据请求
    IAsyncReader::SyncRead //异步读
    IAsyncReader::SyncReadAligned //异步对齐读

## Media Event

    long evCode;
    LONG_PTR param1, param2;
    HRESULT hr;
    while (hr = pEvent->GetEvent(&evCode, &param1, &param2, 0), SUCCEEDED(hr))
    {
        switch(evCode) 
        { 
    	// Call application-defined functions for each 
    	// type of event that you want to handle.
        } 
        hr = pEvent->FreeEventParams(evCode, param1, param2);
    }
    
    IMediaEvent::CancelDefaultHandling //取消默认事件处理(使事件上报到应用程序)
    IMediaEvent::RestoreDefaultHandling //恢复默认事件处理
    
    //窗口通知
    #define WM_GRAPHNOTIFY WM_APP + 1
    pEvent->SetNotifyWindow((OAHWND)g_hwnd, WM_GRAPHNOTIFY, 0);
    
    //事件信号
    HANDLE  hEvent; 
    long    evCode, param1, param2;
    BOOLEAN bDone = FALSE;
    HRESULT hr = S_OK;
    hr = pEvent->GetEventHandle((OAEVENT*)&hEvent);
    while(!bDone) 
    {
        if (WAIT_OBJECT_0 == WaitForSingleObject(hEvent, 100))
        { 
    	while (S_OK == pEvent->GetEvent(&evCode, &param1, &param2, 0)) 
    	{
    	    printf("Event code: %#04x\n Params: %d, %d\n", evCode, param1, param2);
    	    pEvent->FreeEventParams(evCode, param1, param2);
    	    bDone = (EC_COMPLETE == evCode);
    	}
        }
    } 

## Window Mode

    //获得IVideoWindow接口
    IVideoWindow* pVidWin = NULL;
    pGraph->QueryInterface(IID_IVideoWindow, (void**)&pVidWin);
    
    //设置父窗口
    pVidWin->put_Owner((OAHWND)hwnd);
    
    //设置窗口样式
    pVidWin->put_WindowStyle(WS_CHILD | WS_CLIPSIBLINGS);
    
    //定位视频:拉伸视频以适应父窗口工作区
    RECT rc;
    GetClientRect(hwnd, &rc);
    pVidWin->SetWindowPosition(0, 0, rc.right, rc.bottom);
    
    //暂停时移动窗口事件响应
    case WM_MOVE:
        pVidWin->NotifyOwnerMessage((OAHWND)hWnd, msg, wParam, lParam);
        break;
    
    //清理
    pControl->Stop(); 
    pVidWin->put_Visible(OAFALSE);
    pVidWin->put_Owner(NULL);  


## None-Window Mode

适用于VMR

    HRESULT InitWindowlessVMR( 
        HWND hwndApp,                  // 显示窗口
        IGraphBuilder* pGraph,         // Filter Graph管理器
        IVMRWindowlessControl** ppWc   // 返回的VMR指针.
        ) 
    { 
        if (!pGraph || !ppWc) 
        {
    	return E_POINTER;
        }
        IBaseFilter* pVmr = NULL; 
        IVMRWindowlessControl* pWc = NULL; 
        // 创建VMR
        HRESULT hr = CoCreateInstance(CLSID_VideoMixingRenderer, NULL, 
    	CLSCTX_INPROC, IID_IBaseFilter, (void**)&pVmr); 
        if (FAILED(hr))
        {
    	return hr;
        }
    
        // 将VMR添加到Graph
        hr = pGraph->AddFilter(pVmr, L"Video Mixing Renderer"); 
        if (FAILED(hr)) 
        {
    	pVmr->Release();
    	return hr;
        }
    
        // 设置渲染模式
        IVMRFilterConfig* pConfig; 
        hr = pVmr->QueryInterface(IID_IVMRFilterConfig, (void**)&pConfig); 
        if (SUCCEEDED(hr)) 
        { 
    	hr = pConfig->SetRenderingMode(VMRMode_Windowless); 
    	pConfig->Release(); 
        }
        if (SUCCEEDED(hr))
        {
    	// 设置窗口
    	hr = pVmr->QueryInterface(IID_IVMRWindowlessControl, (void**)&pWc);
    	if( SUCCEEDED(hr)) 
    	{ 
    	    hr = pWc->SetVideoClippingWindow(hwndApp); 
    	    if (SUCCEEDED(hr))
    	    {
    		*ppWc = pWc; 
    	    }
    	    else
    	    {
    		pWc->Release();
    	    }
    	} 
        } 
        pVmr->Release(); 
        return hr; 
    }
    
    //调用上述函数
    IVMRWindowlessControl *pWc = NULL;
    hr = InitWindowlessVMR(hwnd, pGraph, &g_pWc);
    if (SUCCEEDED(hr))
    {
        //创建graph,如从文件渲染
        pGraph->RenderFile(wszMyFileName, 0);
        //释放VMR接口
        pWc->Release();
    }
    
    
    // 获取源大小
    long lWidth, lHeight; 
    HRESULT hr = g_pWc->GetNativeVideoSize(&lWidth, &lHeight, NULL, NULL); 
    if (SUCCEEDED(hr))
    {
        RECT rcSrc, rcDest; 
        // 设置源矩形
        SetRect(&rcSrc, 0, 0, lWidth, lHeight); 
    
        // 获取窗口客户区
        GetClientRect(hwnd, &rcDest); 
        // 设置目的矩形
        SetRect(&rcDest, 0, 0, rcDest.right, rcDest.bottom); 
    
        // 设置渲染位置
        hr = g_pWc->SetVideoPosition(&rcSrc, &rcDest); 
    }
    
    //响应画图消息
    void OnPaint(HWND hwnd) 
    { 
        PAINTSTRUCT ps; 
        HDC         hdc; 
        RECT        rcClient; 
        GetClientRect(hwnd, &rcClient); 
        hdc = BeginPaint(hwnd, &ps); 
        if (g_pWc != NULL) 
        { 
    	// 窗口绘制区域
    	HRGN rgnClient = CreateRectRgnIndirect(&rcClient); 
    	HRGN rgnVideo  = CreateRectRgnIndirect(&g_rcDest);  
    	CombineRgn(rgnClient, rgnClient, rgnVideo, RGN_DIFF);  
    
    	// 绘制窗口
    	HBRUSH hbr = GetSysColorBrush(COLOR_BTNFACE); 
    	FillRgn(hdc, rgnClient, hbr); 
    
    	// 清理
    	DeleteObject(hbr); 
    	DeleteObject(rgnClient); 
    	DeleteObject(rgnVideo); 
    
    	// 绘制视频
    	HRESULT hr = g_pWc->RepaintVideo(hwnd, hdc);  
        } 
        else //没有视频则绘制完整的窗口
        { 
    	FillRect(hdc, &rc2, (HBRUSH)(COLOR_BTNFACE + 1)); 
        } 
        EndPaint(hwnd, &ps); 
    }

## System Device Enumerator

    // 创建系统设备枚举器
    HRESULT hr;
    ICreateDevEnum*pSysDevEnum = NULL;
    hr = CoCreateInstance(CLSID_SystemDeviceEnum, NULL, CLSCTX_INPROC_SERVER,
        IID_ICreateDevEnum, (void**)&pSysDevEnum);
    if (FAILED(hr))
    {
        return hr;
    }
    
    // 获取某一类设备的枚举器
    IEnumMoniker *pEnumCat = NULL;
    hr = pSysDevEnum->CreateClassEnumerator(CLSID_VideoCompressorCategory, &pEnumCat, 0);
    
    if (hr == S_OK) 
    {
        // 枚举设备
        IMoniker *pMoniker = NULL;
        ULONG cFetched;
        while(pEnumCat->Next(1, &pMoniker, &cFetched) == S_OK)
        {
    	IPropertyBag *pPropBag;
    	hr = pMoniker->BindToStorage(0, 0, IID_IPropertyBag, 
    	    (void **)&pPropBag);
    	if (SUCCEEDED(hr))
    	{
    	    // 获取可读设备名称
    	    VARIANT varName;
    	    VariantInit(&varName);
    	    hr = pPropBag->Read(L"FriendlyName", &varName, 0);
    	    if (SUCCEEDED(hr))
    	    {
    		// 显示名称(略)
    	    }
    	    VariantClear(&varName);
    
    	    // 由设备创建Filter实例
    	    IBaseFilter *pFilter;
    	    hr = pMoniker->BindToObject(NULL, NULL, IID_IBaseFilter,
    		(void**)&pFilter);
    
    	    // 将Filter加入到Graph(略)
    
    	    pPropBag->Release();
    	}
    	pMoniker->Release();
        }
        pEnumCat->Release();
    }
    pSysDevEnum->Release();
    
    //从设备添加源Filter到Graph
    IBaseFilter *pSource = NULL;
    IMoniker *pMoniker = NULL;
    // 枚举设备(略)
    
    
    // 使用moniker创建绑定上下文
    IBindCtx pContext=0;
    hr = CreateBindCtx(0, &pContext);
    if (SUCCEEDED(hr))
    {
    // 请求IFilterGraph2接口
    IFilterGraph2 pFG2 = NULL;
    hr = pGraph->QueryInterface(IID_IFilterGraph2, (void)&pFG2);
    if (SUCCEEDED(hr))
    {
    // 创建源Filter
    hr = pFG2->AddSourceFilterForMoniker(pMoniker, pContext,
    L"Source", &pSource);
    pFG2->Release();
    }
    pContext->Release();
    }
    pMoniker->Release();

## Filter Mapper

如果需要查找支持特定媒体类型组合的筛选器，但不属于明确类别，可能需要使用筛选器映射器

    IFilterMapper2 *pMapper = NULL;
    IEnumMoniker *pEnum = NULL;
    
    hr = CoCreateInstance(CLSID_FilterMapper2, 
        NULL, CLSCTX_INPROC, IID_IFilterMapper2, 
        (void **) &pMapper);
    
    if (FAILED(hr))
    {
        // 错误处理
    }
    
    GUID arrayInTypes[2];
    arrayInTypes[0] = MEDIATYPE_Video;
    arrayInTypes[1] = MEDIASUBTYPE_dvsd;
    
    //枚举属性匹配的Fliter
    hr = pMapper->EnumMatchingFilters(
    	&pEnum,
    	0,                  // Reserved.
    	TRUE,               // Use exact match?
    	MERIT_DO_NOT_USE+1, // Minimum merit.
    	TRUE,               // At least one input pin?
    	1,                  // Number of major type/subtype pairs for input.
    	arrayInTypes,       // Array of major type/subtype pairs for input.
    	NULL,               // Input medium.
    	NULL,               // Input pin category.
    	FALSE,              // Must be a renderer?
    	TRUE,               // At least one output pin?
    	0,                  // Number of major type/subtype pairs for output.
    	NULL,               // Array of major type/subtype pairs for output.
    	NULL,               // Output medium.
    	NULL);              // Output pin category.
    
    // 枚举moniker.
    IMoniker *pMoniker;
    ULONG cFetched;
    while (pEnum->Next(1, &pMoniker, &cFetched) == S_OK)
    {
        IPropertyBag *pPropBag = NULL;
        hr = pMoniker->BindToStorage(0, 0, IID_IPropertyBag, 
           (void **)&pPropBag);
    
        if (SUCCEEDED(hr))
        {
    	// 提取设备名称
    	VARIANT varName;
    	VariantInit(&varName);
    	hr = pPropBag->Read(L"FriendlyName", &varName, 0);
    	if (SUCCEEDED(hr))
    	{
    	    // 显示设备名称
    	}
    	VariantClear(&varName);
    
    	// 由monitor创建Filter实例
    	IBaseFilter*pFilter;
    	hr = pMoniker->BindToObject(NULL, NULL, IID_IBaseFilter, (void**)&pFilter);
    
    	// 将Filter添加到Graph
    
    	pPropBag->Release();
        }
        pMoniker->Release();
    }
    
    // Clean up.
    pMapper->Release();
    pEnum->Release();

## Enumerate Fliters in Graph

    HRESULT EnumFilters (IFilterGraph *pGraph) 
    {
        IEnumFilters *pEnum = NULL;
        IBaseFilter *pFilter;
        ULONG cFetched;
    
        HRESULT hr = pGraph->EnumFilters(&pEnum);
        if (FAILED(hr)) return hr;
    
        while(pEnum->Next(1, &pFilter, &cFetched) == S_OK)
        {
    	FILTER_INFO FilterInfo;
    	hr = pFilter->QueryFilterInfo(&FilterInfo);
    	if (FAILED(hr))
    	{
    	    MessageBox(NULL, TEXT("Could not get the filter info"),
    		TEXT("Error"), MB_OK | MB_ICONERROR);
    	    continue;  // Maybe the next one will work.
    	}
    
    #ifdef UNICODE
    	MessageBox(NULL, FilterInfo.achName, TEXT("Filter Name"), MB_OK);
    #else
    	char szName[MAX_FILTER_NAME];
    	int cch = WideCharToMultiByte(CP_ACP, 0, FilterInfo.achName,
    	    MAX_FILTER_NAME, szName, MAX_FILTER_NAME, 0, 0);
    	if (cch > 0)
    	    MessageBox(NULL, szName, TEXT("Filter Name"), MB_OK);
    #endif
    
    	// 必须释放对FilterManager的引用计数指针
    	if (FilterInfo.pGraph != NULL)
    	{
    	    FilterInfo.pGraph->Release();
    	}
    	pFilter->Release();
        }
    
        pEnum->Release();
        return S_OK;
    }

## Enumerate Pins in Graph

    HRESULT GetPin(IBaseFilter *pFilter, PIN_DIRECTION PinDir, IPin **ppPin)
    {
        IEnumPins  *pEnum = NULL;
        IPin       *pPin = NULL;
        HRESULT    hr;
    
        if (ppPin == NULL)
        {
    	return E_POINTER;
        }
    
        hr = pFilter->EnumPins(&pEnum);
        if (FAILED(hr))
        {
    	return hr;
        }
        while(pEnum->Next(1, &pPin, 0) == S_OK)
        {
    	PIN_DIRECTION PinDirThis;
    	hr = pPin->QueryDirection(&PinDirThis);
    	if (FAILED(hr))
    	{
    	    pPin->Release();
    	    pEnum->Release();
    	    return hr;
    	}
    	if (PinDir == PinDirThis)
    	{
    	    // 找到一个Pin
    	    *ppPin = pPin;
    	    pEnum->Release();
    	    return S_OK;
    	}
    	// 释放Pin
    	pPin->Release();
        }
        // 没有找到Pin
        pEnum->Release();
        return E_FAIL;  
    }

## Enumerate Media Type of A Pin

    HRESULT GetPinMediaType(
        IPin *pPin,             // Pin指针
        REFGUID majorType,      // 主类型
        REFGUID subType,        // 子类型
        REFGUID formatType,     // 格式类型
        AM_MEDIA_TYPE **ppmt    // Media Type指针,用于获取格式参数详情,可为NULL
        )
    {
        *ppmt = NULL;
    
        IEnumMediaTypes *pEnum = NULL;
        AM_MEDIA_TYPE *pmt = NULL;
        BOOL bFound = FALSE;
    
        HRESULT hr = pPin->EnumMediaTypes(&pEnum);
        if (FAILED(hr))
        {
    	return hr;
        }
    
        while (hr = pEnum->Next(1, &pmt, NULL), hr == S_OK)
        {
    	if ((majorType==GUID_NULL) || (majorType==pmt->majortype))
    	{
    	    if ((subType==GUID_NULL) || (subType==pmt->subtype))
    	    {
    		if ((formatType==GUID_NULL) || (formatType==pmt->formattype))
    		{
    		    // Found a match. 
    		    if (ppmt)
    		    {
    			*ppmt = pmt;  // 返回类型匹配的Media Type
    		    }
    		    else
    		    {
    			_DeleteMediaType(pmt);
    		    }
    		    bFound = TRUE;
    		    break;
    		}
    	    }
    	}
    	_DeleteMediaType(pmt);
        }
    
        SafeRelease(&pEnum);
        if (SUCCEEDED(hr))
        {
    	if (!bFound)
    	{
    	    hr = VFW_E_NOT_FOUND;
    	}
        }
        return hr;
    }

## Add A Filter to Graph by CLSID

    HRESULT AddFilterByCLSID(
        IGraphBuilder *pGraph,      // Filter Graph Manager指针
        REFGUID clsid,              // Filter的CLSID
        IBaseFilter **ppF,          // Filter的指针
        LPCWSTR wszName             // Filter的名称,可为NULL
        )
    {
        *ppF = 0;
    
        IBaseFilter *pFilter = NULL;
    
        HRESULT hr = CoCreateInstance(clsid, NULL, CLSCTX_INPROC_SERVER, 
    	IID_PPV_ARGS(&pFilter));
        if (FAILED(hr))
        {
    	goto done;
        }
    
        hr = pGraph->AddFilter(pFilter, wszName);
        if (FAILED(hr))
        {
    	goto done;
        }
    
        *ppF = pFilter;
        (*ppF)->AddRef();
    
    done:
        SafeRelease(&pFilter);
        return hr;
    }
    
    //添加一个AVI Mux
    IBaseFilter *pMux;
    hr = AddFilterByCLSID(pGraph, CLSID_AviDest, L"AVI Mux", &pMux, NULL); 
    if (SUCCEEDED(hr))
    {
        /* ... */
       pMux->Release();
    }

## Find Unconnected Pins

    //Pin是否连接
    HRESULT IsPinConnected(IPin *pPin, BOOL *pResult)
    {
        IPin *pTmp = NULL;
        HRESULT hr = pPin->ConnectedTo(&pTmp);
        if (SUCCEEDED(hr))
        {
    	*pResult = TRUE;
        }
        else if (hr == VFW_E_NOT_CONNECTED)
        {
    	// The pin is not connected. This is not an error for our purposes.
    	*pResult = FALSE;
    	hr = S_OK;
        }
    
        SafeRelease(&pTmp);
        return hr;
    }
    
    // 特定方向上的Pin
    HRESULT IsPinDirection(IPin *pPin, PIN_DIRECTION dir, BOOL *pResult)
    {
        PIN_DIRECTION pinDir;
        HRESULT hr = pPin->QueryDirection(&pinDir);
        if (SUCCEEDED(hr))
        {
    	*pResult = (pinDir == dir);
        }
        return hr;
    }
    
    // 匹配Pin的链接状态
    HRESULT MatchPin(IPin *pPin, PIN_DIRECTION direction, BOOL bShouldBeConnected, BOOL *pResult)
    {
        assert(pResult != NULL);
    
        BOOL bMatch = FALSE;
        BOOL bIsConnected = FALSE;
    
        HRESULT hr = IsPinConnected(pPin, &bIsConnected);
        if (SUCCEEDED(hr))
        {
    	if (bIsConnected == bShouldBeConnected)
    	{
    	    hr = IsPinDirection(pPin, direction, &bMatch);
    	}
        }
    
        if (SUCCEEDED(hr))
        {
    	*pResult = bMatch;
        }
        return hr;
    }
    
    // 返回第一个未连接的Pin
    HRESULT FindUnconnectedPin(IBaseFilter *pFilter, PIN_DIRECTION PinDir, IPin **ppPin)
    {
        IEnumPins *pEnum = NULL;
        IPin *pPin = NULL;
        BOOL bFound = FALSE;
    
        HRESULT hr = pFilter->EnumPins(&pEnum);
        if (FAILED(hr))
        {
    	goto done;
        }
    
        while (S_OK == pEnum->Next(1, &pPin, NULL))
        {
    	hr = MatchPin(pPin, PinDir, FALSE, &bFound);
    	if (FAILED(hr))
    	{
    	    goto done;
    	}
    	if (bFound)
    	{
    	    *ppPin = pPin;
    	    (*ppPin)->AddRef();
    	    break;
    	}
    	SafeRelease(&pPin);
        }
    
        if (!bFound)
        {
    	hr = VFW_E_NOT_FOUND;
        }
    
    done:
        SafeRelease(&pPin);
        SafeRelease(&pEnum);
        return hr;
    }

## Connect Two Filters

    // 将输出Pin连接到Filter
    HRESULT ConnectFilters(
        IGraphBuilder *pGraph, // Filter Graph Manager.
        IPin *pOut,            // 上游Filter的输出Pin
        IBaseFilter *pDest)    // 下游Filter
    {
        IPin *pIn = NULL;
    
        // 在下游Filter上找一个输入Pin
        HRESULT hr = FindUnconnectedPin(pDest, PINDIR_INPUT, &pIn);
        if (SUCCEEDED(hr))
        {
    	// 尝试连接
    	hr = pGraph->Connect(pOut, pIn);
    	pIn->Release();
        }
        return hr;
    }
    
    // 将输入Pin连接到Filter
    HRESULT ConnectFilters(IGraphBuilder *pGraph, IBaseFilter *pSrc, IPin *pIn)
    {
        IPin *pOut = NULL;
    
        // 在上游Filter查找输出Pin
        HRESULT hr = FindUnconnectedPin(pSrc, PINDIR_OUTPUT, &pOut);
        if (SUCCEEDED(hr))
        {
    	// 尝试连接
    	hr = pGraph->Connect(pOut, pIn);
    	pOut->Release();
        }
        return hr;
    }
    
    // 连接两个Filter
    HRESULT ConnectFilters(IGraphBuilder *pGraph, IBaseFilter *pSrc, IBaseFilter *pDest)
    {
        IPin *pOut = NULL;
    
        // 在第一个Filter上查找输出Pin
        HRESULT hr = FindUnconnectedPin(pSrc, PINDIR_OUTPUT, &pOut);
        if (SUCCEEDED(hr))
        {
    	hr = ConnectFilters(pGraph, pOut, pDest);
    	pOut->Release();
        }
        return hr;
    }

## Find Filter/Pin Interface

    // 在Filter上查找接口
    HRESULT FindFilterInterface(
        IGraphBuilder *pGraph, // Filter Graph Manager指针
        REFGUID iid,           // 接口IID
        void **ppUnk)          // 接口指针
    {
        if (!pGraph || !ppUnk) return E_POINTER;
    
        HRESULT hr = E_FAIL;
        IEnumFilters *pEnum = NULL;
        IBaseFilter *pF = NULL;
        if (FAILED(pGraph->EnumFilters(&pEnum)))
        {
    	return E_FAIL;
        }
        // 在每一个Filter上查找接口
        while (S_OK == pEnum->Next(1, &pF, 0))
        {
    	hr = pF->QueryInterface(iid, ppUnk);
    	pF->Release();
    	if (SUCCEEDED(hr))
    	{
    	    break;
    	}
        }
        pEnum->Release();
        return hr;
    }
    
    // 在Pin上查找接口
    HRESULT FindPinInterface(
        IBaseFilter *pFilter,  // Filter指针
        REFGUID iid,           // 接口IID
        void **ppUnk)          // 接口指针
    {
        if (!pFilter || !ppUnk) return E_POINTER;
    
        HRESULT hr = E_FAIL;
        IEnumPins *pEnum = 0;
        if (FAILED(pFilter->EnumPins(&pEnum)))
        {
    	return E_FAIL;
        }
        // 在每一个Pin上查找接口
        IPin *pPin = 0;
        while (S_OK == pEnum->Next(1, &pPin, 0))
        {
    	hr = pPin->QueryInterface(iid, ppUnk);
    	pPin->Release();
    	if (SUCCEEDED(hr))
    	{
    	    break;
    	}
        }
        pEnum->Release();
        return hr;
    }
    
    // 通用查找接口函数
    HRESULT FindInterfaceAnywhere(
        IGraphBuilder *pGraph, 
        REFGUID iid, 
        void **ppUnk)
    {
        if (!pGraph || !ppUnk) return E_POINTER;
        HRESULT hr = E_FAIL;
        IEnumFilters *pEnum = 0;
        if (FAILED(pGraph->EnumFilters(&pEnum)))
        {
    	return E_FAIL;
        }
        // 循环每一个Filter
        IBaseFilter *pF = 0;
        while (S_OK == pEnum->Next(1, &pF, 0))
        {
    	hr = pF->QueryInterface(iid, ppUnk);
    	if (FAILED(hr))
    	{
    	    // 查找Filter的Pin
    	    hr = FindPinInterface(pF, iid, ppUnk);
    	}
    	pF->Release();
    	if (SUCCEEDED(hr))
    	{
    	    break;
    	}
        }
        pEnum->Release();
        return hr;
    }

### Find Peer Filter

    // 获取上游或下游的Filter
    HRESULT GetNextFilter(
        IBaseFilter *pFilter, // 当前Filter指针
        PIN_DIRECTION Dir,    // 获取方向
        IBaseFilter **ppNext) // 下一个Fliter指针
    {
        if (!pFilter || !ppNext) return E_POINTER;
    
        IEnumPins *pEnum = 0;
        IPin *pPin = 0;
        HRESULT hr = pFilter->EnumPins(&pEnum);
        if (FAILED(hr)) return hr;
        while (S_OK == pEnum->Next(1, &pPin, 0))
        {
    	// 检查Pin的方向
    	PIN_DIRECTION ThisPinDir;
    	hr = pPin->QueryDirection(&ThisPinDir);
    	if (FAILED(hr))
    	{
    	    // 异常处理
    	    hr = E_UNEXPECTED;
    	    pPin->Release();
    	    break;
    	}
    	if (ThisPinDir == Dir)
    	{
    	    // 检查Pin是否连接了其他Pin
    	    IPin *pPinNext = 0;
    	    hr = pPin->ConnectedTo(&pPinNext);
    	    if (SUCCEEDED(hr))
    	    {
    		// 获取对应Pin的Filter
    		PIN_INFO PinInfo;
    		hr = pPinNext->QueryPinInfo(&PinInfo);
    		pPinNext->Release();
    		pPin->Release();
    		pEnum->Release();
    		if (FAILED(hr) || (PinInfo.pFilter == NULL))
    		{
    		    // 异常处理
    		    return E_UNEXPECTED;
    		}
    		// 找到Filter
    		*ppNext = PinInfo.pFilter; // 释放PinInfo中的引用
    		return S_OK;
    	    }
    	}
    	pPin->Release();
        }
        pEnum->Release();
        // 未找到Filter
        return E_FAIL;
    }
    
    //调用代码
    IBaseFilter *pF; // Filter指针
    IBaseFilter *pUpstream = NULL;
    if (SUCCEEDED(GetNextFilter(pF, PINDIR_INPUT, &pUpstream)))
    {
        // 做一些处理
        pUpstream->Release();
    }
    
    #include <streams.h>  // DirectShow基类
    // 定义一个Filter列表
    typedef CGenericList<IBaseFilter> CFilterList;
    
    // 前向声明, 添加一个Filter到列表
    void AddFilterUnique(CFilterList &FilterList, IBaseFilter *pNew);
    
    // 查找直连的上游或下游Filter.
    HRESULT GetPeerFilters(
        IBaseFilter *pFilter, // 当前Filter指针
        PIN_DIRECTION Dir,    // 查找方向
        CFilterList &FilterList)  // 查找结果列表
    {
        if (!pFilter) return E_POINTER;
    
        IEnumPins *pEnum = 0;
        IPin *pPin = 0;
        HRESULT hr = pFilter->EnumPins(&pEnum);
        if (FAILED(hr)) return hr;
        while (S_OK == pEnum->Next(1, &pPin, 0))
        {
    	// 检查查找方向
    	PIN_DIRECTION ThisPinDir;
    	hr = pPin->QueryDirection(&ThisPinDir);
    	if (FAILED(hr))
    	{
    	    // 异常处理
    	    hr = E_UNEXPECTED;
    	    pPin->Release();
    	    break;
    	}
    	if (ThisPinDir == Dir)
    	{
    	    // 检查Pin是否连接
    	    IPin *pPinNext = 0;
    	    hr = pPin->ConnectedTo(&pPinNext);
    	    if (SUCCEEDED(hr))
    	    {
    		// 获取连接的Filter
    		PIN_INFO PinInfo;
    		hr = pPinNext->QueryPinInfo(&PinInfo);
    		pPinNext->Release();
    		if (FAILED(hr) || (PinInfo.pFilter == NULL))
    		{
    		    // 异常处理
    		    pPin->Release();
    		    pEnum->Release();
    		    return E_UNEXPECTED;
    		}
    		// 将Filter插入到列表
    		AddFilterUnique(FilterList, PinInfo.pFilter);
    		PinInfo.pFilter->Release();
    	    }
    	}
    	pPin->Release();
        }
        pEnum->Release();
        return S_OK;
    }
    void AddFilterUnique(CFilterList &FilterList, IBaseFilter *pNew)
    {
        if (pNew == NULL) return;
    
        POSITION pos = FilterList.GetHeadPosition();
        while (pos)
        {
    	IBaseFilter *pF = FilterList.GetNext(pos);
    	if (IsEqualObject(pF, pNew))
    	{
    	    return;
    	}
        }
        pNew->AddRef();  // 调用者负责是否列表中的资源
        FilterList.AddTail(pNew);
    }
    
    //调用代码
    IBaseFilter *pF; // Filter指针
    CFilterList FList(NAME("MyList"));  // Filter列表
    hr = GetPeerFilters(pF, PINDIR_OUTPUT, FList);
    if (SUCCEEDED(hr))
    {
        POSITION pos = FList.GetHeadPosition();
        while (pos)
        {
    	IBaseFilter *pDownstream = FList.GetNext(pos);
    	pDownstream->Release();
        }
    }

## Delete All Filters in Graph

    // 停止数据流
    pControl->Stop();
    
    // 枚举Graph中的Filter
    IEnumFilters *pEnum = NULL;
    HRESULT hr = pGraph->EnumFilters(&pEnum);
    if (SUCCEEDED(hr))
    {
        IBaseFilter *pFilter = NULL;
        while (S_OK == pEnum->Next(1, &pFilter, NULL))
         {
    	 // 移除Filter
    	 pGraph->RemoveFilter(pFilter);
    	 // 重置枚举器
    	 pEnum->Reset();
    	 pFilter->Release();
        }
        pEnum->Release();
    }

## Use Capture Graph Builder

      IGraphBuilder *pGraph = NULL;
          ICaptureGraphBuilder2 *pBuilder = NULL;
    
          // 创建Filter Graph Manager.
          HRESULT hr =  CoCreateInstance(CLSID_FilterGraph, NULL,
    	  CLSCTX_INPROC_SERVER, IID_IGraphBuilder, (void **)&pGraph);
    
          if (SUCCEEDED(hr))
          {
    	  // 创建Capture Graph Builder.
    	  hr = CoCreateInstance(CLSID_CaptureGraphBuilder2, NULL,
    	      CLSCTX_INPROC_SERVER, IID_ICaptureGraphBuilder2, 
    	      (void **)&pBuilder);
    	  if (SUCCEEDED(hr))
    	  {
    	      pBuilder->SetFiltergraph(pGraph);
    	  }
          };
    
        // 捕获到文件,第一个参数时Pin的属性集,第二个参数是MediaType,后面是要连接的Filter
        pBuilder->RenderStream(&PIN_CATEGORY_CAPTURE, NULL, pCapFilter, NULL, pMux);
        // 系统自动选择预览Render
        pBuilder->RenderStream(&PIN_CATEGORY_PREVIEW, NULL, pCapFilter, NULL, NULL);
    
      //查找接口
      IAMStreamConfig *pConfig = NULL;
      HRESULT hr = pBuild->FindInterface(
          &PIN_CATEGORY_PREVIEW, 
          &MEDIATYPE_Video,
          pVCap, 
          IID_IAMStreamConfig, 
          (void**)&pConfig
      );
      if (SUCCESSFUL(hr))
      {
          /* ... */
          pConfig->Release();
      }
    
    //查找Pin
    IPin *pPin = NULL;
    hr = pBuild->FindPin(
        pCap,                   // Filter指针
        PINDIR_OUTPUT,          // 输出Pin
        &PIN_CATEGORY_PREVIEW,  // 预览类型Pin.
        &MEDIATYPE_Video,       // 视频类型Pin.
        TRUE,                   // 已连接. 
        0,                      // 查找结果索引
        &pPin);                 // Pin指针
    if (SUCCESSFUL(hr))
    {
        /* ... */
        pPin->Release();
    }

## Media Seeking

如果流支持Seeking,那么可以:

-   在流中定位到任意位置。
-   检索流的持续时间。
-   检索流中的当前位置。
-   反向播放。

```
    DWORD dwCap = 0;
    HRESULT hr = pSeek->GetCapabilities(&dwCap);
    if (AM_SEEKING_CanSeekAbsolute & dwCap)
    {
        // 将Graph定位到一个绝对位置
    }
```

### Set Position & Get Position

    #define ONE_SECOND 10000000
    REFERENCE_TIME rtNow  = 2 * ONE_SECOND, 
    	       rtStop = 5 * ONE_SECOND;
    
    //设置绝对位置
    hr = pSeek->SetPositions(
        &rtNow,  AM_SEEKING_AbsolutePositioning, 
        &rtStop, AM_SEEKING_AbsolutePositioning
        );
    
    //设置相对位置
    REFERENCE_TIME rtNow = 10 * ONE_SECOND;
    hr = pSeek->SetPositions(
        &rtNow, AM_SEEKING_RelativePositioning, 
        NULL, AM_SEEKING_NoPositioning
        );


<a id="orgaa4457d"></a>

### SetRate

    pSeek->SetRate(2.0)


<a id="orgb618196"></a>

### Set Time Format

    hr = pSeek->IsFormatSupported(&TIME_FORMAT_FRAME);
    if (hr == S_OK)
    {
        // 设置时间格式为帧
        hr = pSeek->SetTimeFormat(&TIME_FORMAT_FRAME);
        if (SUCCEEDED(hr))
        {
    	// 设置位置到第20帧
    	LONGLONG rtNow = 20;
    	hr = pSeek->SetPositions(
    	    &rtNow, AM_SEEKING_AbsolutePositioning, 
    	    0, AM_SEEKING_NoPositioning);
        }
    }

## Seeking Slider

    void InitSlider(HWND hwnd) 
    {
        // 初始化Slider范围,直到用户打开文件前都不可用
        hScroll = GetDlgItem(hwnd, IDC_SLIDER1);
        EnableWindow(hScroll, FALSE);
        SendMessage(hScroll, TBM_SETRANGE, TRUE, MAKELONG(0, 100));
    }
    IMediaSeeking *g_pSeek = 0;
    hr = pGraph->QueryInterface(IID_IMediaSeeking, (void**)&g_pSeek);
    
    // 判断源是否支持Seeking
    BOOL  bCanSeek = FALSE;
    DWORD caps = AM_SEEKING_CanSeekAbsolute | AM_SEEKING_CanGetDuration; 
    bCanSeek = (S_OK == pSeek->CheckCapabilities(&caps));
    if (bCanSeek)
    {
        // 使能Slider
        EnableWindow(hScroll, TRUE);
    
        // 获取视频长度
        pSeek->GetDuration(&g_rtTotalTime);
    }
    
    //开始播放
    void StartPlayback(HWND hwnd) 
    {
        pControl->Run();
        if (bCanSeek)
        {
    	StopTimer(); // 停止定时器
    	nTimerID = SetTimer(hwnd, IDT_TIMER1, TICK_FREQ, (TIMERPROC)NULL);
    	if (nTimerID == 0)
    	{
    	    /* 错误处理 */
    	}
        }
    }
    
    //停止定时器
    void StopTimer() 
    {
        if (wTimerID != 0)
        {
    	KillTimer(g_hwnd, wTimerID);
    	wTimerID = 0;
        }
    }
    
    //定时器事件处理
    case WM_TIMER:
        if (wParam == IDT_TIMER1)
        {
    	// Timer should not be running unless we really can seek.
    	ASSERT(bCanSeek == TRUE);
    
    	REFERENCE_TIME timeNow;
    	if (SUCCEEDED(pSeek->GetCurrentPosition(&timeNow)))
    	{
    	    long sliderTick = (long)((timeNow * 100) / g_rtTotalTime);
    	    SendMessage( hScroll, TBM_SETPOS, TRUE, sliderTick );
    	}
        }
        break;
    
    //拖动Slider事件处理
    static OAFilterState state;
    static BOOL bStartOfScroll = TRUE;
    
    case WM_HSCROLL:
        short int userReq = LOWORD(wParam);
        if (userReq == TB_ENDTRACK || userReq == TB_THUMBTRACK)
        {
    	DWORD dwPosition  = SendMessage(hTrackbar, TBM_GETPOS, 0, 0);
    	// 拖动时暂停视频
    	if (bStartOfScroll) 
    	{
    	    pControl->GetState(10, &state);
    	    bStartOfScroll = FALSE;
    	    pControl->Pause();
    	}
    	// 持续更新视频位置
    	REFERENCE_TIME newTime = (g_rtTotalTime/100) * dwPosition;
    	pSeek->SetPositions(&newTime, AM_SEEKING_AbsolutePositioning,
    	    NULL, AM_SEEKING_NoPositioning);
    
    	// 恢复播放状态
    	if (userReq == TB_ENDTRACK)
    	{
    	    if (state == State_Stopped)
    		pControl->Stop();
    	    else if (state == State_Running) 
    		pControl->Run();
    	    bStartOfScroll = TRUE;
    	}
        }
    }

## Set Synchronous Source

    IGraphBuilder *pGraph = 0;
    IReferenceClock *pClock = 0;
    
    CoCreateInstance(CLSID_FilterGraph, NULL, CLSCTX_INPROC_SERVER, 
        IID_IGraphBuilder, (void **)&pGraph);
    
    // 创建Graph
    pGraph->RenderFile(L"C:\\Example.avi", 0);
    
    // 创建时钟
    hr = CreateMyPrivateClock(&pClock);
    if (SUCCEEDED(hr))
    {
        // 设置时钟
        IMediaFilter *pMediaFilter = 0;
        pGraph->QueryInterface(IID_IMediaFilter, (void**)&pMediaFilter);
        pMediaFilter->SetSyncSource(pClock);
        pClock->Release();
        pMediaFilter->Release();
    }

