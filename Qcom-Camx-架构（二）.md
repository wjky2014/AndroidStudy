#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Android Hal3回顾
>二、Qcom Hal3 CamX架构
>三、Qcom Hal3 Camx 重点
 


## 一、 Android Hal3回顾

[Camera HAL3学习](https://www.cnblogs.com/tsts/p/9314924.html)
![](https://upload-images.jianshu.io/upload_images/2118860-83e52f9fca3b087c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ ](https://upload-images.jianshu.io/upload_images/2118860-d60638c7265ef9db?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HAL层操作简单总结:
1. Framework层发送捕获数据的异步请求。
2. HAL层设备必须按照次序处理请求。对于每个请求，HAL层需要输出元数据和一个或者多个图像数据。
3. 对于请求和结果都需要遵循先进先出的原则；这个数据流将被后续的请求所参考。
4. 对于同一个请求，所有输出数据的时间戳必须相同，以便framework层同步输出数据，如果需要的话。
5. 在请求和结果数据总，所有捕获数据的配置和状态（除了3A处理），都需要封装起来。

## 二、Qcom Hal3 CamX架构

Qcom作为平台厂商会根据谷歌定义的HAL3接口来实现自己的Camera HAL3,新的主流的Qcom Camera HAL3 架构就是CamX了.
Camx的详细过程可参考高通文档:
80-pc212-1_d_qualcomm_spectra_isp_camera_chi_api_reference.pdf
最主要还是要看code,手机厂商对该层代码有自己的改动,可能差别还是有的,具体项目大体框架一致但细节有区别,差异和机型的高通基线也保持一致.

### 2.1 CamX架构总体结构

简单总结下:
![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/2118860-4b6c8ee02a657bd0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  Camx的架构入口为Camx包中的camxhal3entry.cpp,Camx中是高通平台Camx架构的核心跳转及处理业务的代码,一般手机厂商不会去更改,代码目录在vendor/qcom/proprietary/camx/下,编译结果是camera.qcom.so
2.  Camx通过chxentensionInterface调用到chi-cdk 包下的代码,这里面一般是手机厂商自己定制功能的地方,代码目录在vendor/qcom/proprietary/chi-cdk/,编译结果是com.qti.chi.override.so
3.  从这张图中大概知道一个request来业务流程交给camx处理,但会经过chi-cdk进行request定制化重新打包再交给camx实际去执行或和kernel driver层进行交互,camx部分代码即核心流程管控的代码,而chi-cdk正是手机厂商想要实现自己定制化功能的代码地方.

### 2.2 CamX架构中重要的数据结构及关系

缺一张图

Usecase /Session/ Feature /pipeline/ Node

1.  Chi对Camx的操作，需要通过 ExtensionModule 进行操作，因此，camx对外提供的接口扩展需要通过ExtensionModule进行，里面一个重要的变量就是g_chiContextOps。
2.  Camx对Chi的操作，是通过HAL3Module接口的m_ChiAppCallbacks进行的，因此chi里面释放的接口，都会在m_ChiAppCallbacks里面体现
3.  Usecase：基本上与App上的各种模式有一定的对应关系，包括PreviewZSL,VideoLiveShot, SAT(Multicamera), RTB(Reatime Bokeh)，QCFA, Dual, superslowmotionfrc…， 一个usecase包含了realtime和snapshot的session， 需要包含所有的在这个usecase的所有feature的pipeline列表
4.  Feature: 在Usecase下打开某个功能设计的一种架构，通常情况下一个Usecase可以启用各种不同的feature，feture包含了usecase里面的部分pipeline, 一般都是snapshot的才会包含feature进行处理，因此，usecase与feautre的界限并不明显。
5.  Session： 包含了ChxSession，ChiSession和CamxSession。Chxsession是对chi接口里面对CamxSession的封装，Usecase里面创建的Session都是要创建这一个。ChiSession是camx里面的一个部分，是对camxSession的继承和透传。
6.  pipeline-包含单个经过验证的topology的可重用容器。驱动程序通过pipeline来了解所使用的引擎以及数据处理的流程。
7.  Node—camera pipeline 内的逻辑功能块，在单个引擎上执行。node链接在一起构成一个topology。在CHI API的初始版本中，ISP外部的所有节点都是通过CPU代码调用的，调的是native API。

不同机型,产品性能及定位不同,即使基线一样usecase等也有可能不一样,高通这么做给了手机厂商极大的自定义空间,举个某机型例子,UseCase可以场景复用,对应的pipeline也可以不用或复用.
![ ](https://upload-images.jianshu.io/upload_images/2118860-cd07bd7c83d3da6f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.3 CamX操作过程

基本操作,截图自高通文档:
![ ](https://upload-images.jianshu.io/upload_images/2118860-d72bda388723ec49?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ ](https://upload-images.jianshu.io/upload_images/2118860-3c9511d6eba88247?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ ](https://upload-images.jianshu.io/upload_images/2118860-dbae45d843fa34ec?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

详细过程:

#### 2.3.1 Open Camera![ ](https://upload-images.jianshu.io/upload_images/2118860-5df83ba87245e37b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.3.2 ConfigureStream

![ ](https://upload-images.jianshu.io/upload_images/2118860-a3abecd94e9c615a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.3.3 Request & Result

request:
![ ](https://upload-images.jianshu.io/upload_images/2118860-e4070c3eaf7aa529?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

result:

```
一旦底层有事件上传就会走到SyncManagerPollMethod中：
file: vendor/qcom/proprietary/camx/src/csl/hw/camxsyncmanager.cpp
--> VOID* SyncManager::SyncManagerPollMethod(VOID* pPollData)
   |--> rc = poll(fds, 2, -1)   //监听的syncFd有事件上传
   |   |--> VOID* pData[] = {pEv, NULL};    //这里的pEv是包含回调方法的，根据setRepeatingRequest中的分析，这里的回调就是Node::CSLFenceCallback
   |   |--> ioctl(pCtrl->syncFd, VIDIOC_DQEVENT, pEv);  //取出事件
   |   |--> result = pCtrl->pThreadManager->PostJob(pCtrl->hJob, SyncManager::StoppedCbDispatchJob, pData, FALSE, FALSE);   //这里就会调到之前注册的线程并回调SyncManager::CbDispatchJob方法
       file: vendor/qcom/proprietary/camx/src/csl/hw/camxsyncmanager.cpp
   |   |--> VOID* SyncManager::CbDispatchJob(VOID* pData)
   |   |   |--> Utils::Memcpy(&ev, reinterpret_cast<struct v4l2_event*> (pData), sizeof(ev)); 
   |   |   |--> pPayloadData = CAM_SYNC_GET_PAYLOAD_PTR(ev, uint64_t);
   |   |   |--> reinterpret_cast<CSLFenceHandler>(pPayloadData[0]))(reinterpret_cast<VOID* >(pPayloadData[1]), pEvHeader->sync_obj, fenceResult);   //回调Node::CSLFenceCallback
           file: vendor/qcom/proprietary/camx/src/core/camxnode.cpp
   |   |   |--> VOID Node::CSLFenceCallback(...)
   |   |   |   |--> result = pNode->GetThreadManager()->PostJob(pNode->GetJobFamilyHandle(), NULL, &pData[0], FALSE, FALSE) //将工作放到JobFamilyHandle线程去做,回调Node::NodeThreadJobFamilyCb方法
   |   |   |   |   |--> VOID* Node::NodeThreadJobFamilyCb(...)  //因为是另一个线程处理，所以这个以缩进代表异步关系
   |   |   |   |   |   |--> FenceCallbackData* pFenceCallbackData = static_cast<FenceCallbackData*>(pCbData);  
   |   |   |   |   |   |--> NodeFenceHandlerData* pNodeFenceHandlerData = static_cast<NodeFenceHandlerData*>(pFenceCallbackData->pNodePrivateData);
   |   |   |   |   |   |--> pFenceCallbackData->pNode->ProcessFenceCallback(pNodeFenceHandlerData);
   |   |   |   |   |   |   |--> OutputPort* pOutputPort    = pFenceHandlerData->pOutputPort;
   |   |   |   |   |   |   |--> UINT64      requestId      = pFenceHandlerData->requestId;
   |   |   |   |   |   |   |--> UINT        requestIdIndex = requestId % MaxRequestQueueDepth;
   |   |   |   |   |   |   |--> m_pPipeline->NonSinkPortFenceSignaled(&pFenceHandlerData->hFence, pFenceHandlerData->requestId);    //如果是no sink port
                           file: vendor/qcom/proprietary/camx/src/core/camxpipeline.cpp
   |   |   |   |   |   |   |--> VOID Pipeline::NonSinkPortFenceSignaled(...)
   |   |   |   |   |   |   |   |--> m_pDeferredRequestQueue->FenceSignaledCallback(phFence, requestId);
                               file: vendor/qcom/proprietary/camx/src/core/camxdeferredrequestqueue.cpp
   |   |   |   |   |   |   |   |--> VOID DeferredRequestQueue::FenceSignaledCallback(...)
   |   |   |   |   |   |   |   |   |--> UpdateDependency(PropertyIDInvalid, phFence, NULL, requestId, 0, TRUE);
   |   |   |   |   |   |   |   |   |--> DispatchReadyNodes();
   |   |   |   |   |   |   |   |   |   |--> CamxResult result = m_pThreadManager->PostJob(m_hDeferredWorker, NULL, &pData[0], FALSE, FALSE);    //针对所有ready Nodes 循环处理,m_hDefferredWorker 对应的回调是DeferredWorkerWrapper
   |   |   |   |   |   |   |   |   |   |   |--> VOID* DeferredRequestQueue::DeferredWorkerWrapper(VOID* pData)  //异步调用,以缩进表示
   |   |   |   |   |   |   |   |   |   |   |   |--> Dependency* pDependency = reinterpret_cast<Dependency*>(pData);
   |   |   |   |   |   |   |   |   |   |   |   |--> DeferredRequestQueue* pDeferredQueue = pDependency->pInstance;
   |   |   |   |   |   |   |   |   |   |   |   |--> result = pDeferredQueue->DeferredWorkerCore(pDependency);
   |   |   |   |   |   |   |   |   |   |   |   |   |--> pNode->ProcessRequest(&processRequest, pDependency->requestId)
                                                   file: vendor/qcom/proprietary/camx/src/core/camxnode.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |--> CamxResult Node::ProcessRequest(NodeProcessRequestData* pNodeRequestData, UINT64 requestId)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> result = ExecuteProcessRequest(&executeProcessData); //这里暂时以ipe node为例进行分析
                                                       file: vendor/qcom/proprietary/camx/src/hwl/ipe/camxipenode.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> CamxResult IPENode::ExecuteProcessRequest(ExecuteProcessRequestData* pExecuteProcessRequestData)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> GetHwContext()->Submit(GetCSLSession(), m_hDevice, pIQPacket);  //发送设置命令
                                                           file: vendor/qcom/proprietary/camx/src/core/camxhwcontext.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> CamxResult HwContext::Submit(CSLHandle hCSLSession, CSLDeviceHandle hDevice, Packet* pPacket)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> result = CSLSubmit(hCSLSession, hDevice, pPacket->GetMemHandle(), pPacket->GetOffset());
                                                               file: vendor/qcom/proprietary/camx/src/csl/camxcsl.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> CamxResult CSLSubmit(...)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> return pJumpTable->CSLSubmit(hCSL, hDevice, hPacket, offset);  //跳转方法见func_list_camx_chi ⑤ ,接下来我就不分析了
   |   |   |   |   |   |   |--> m_pPipeline->SinkPortFenceSignaled(pOutputPort->sinkTargetStreamId, ...)    //如果是sink port
                           file: vendor/qcom/proprietary/camx/src/core/camxpipeline.cpp
   |   |   |   |   |   |   |--> VOID Pipeline::SinkPortFenceSignaled(...)
   |   |   |   |   |   |   |   |--> ResultsData resultsData = {};
   |   |   |   |   |   |   |   |--> resultsData.pPrivData = pPerRequestInfo->request.pPrivData;
   |   |   |   |   |   |   |   |--> resultsData.type      = CbType::Buffer;
   |   |   |   |   |   |   |   |--> m_pSession->NotifyResult(&resultsData);
                               file: vendor/qcom/proprietary/camx/src/core/camxsession.cpp
   |   |   |   |   |   |   |   |--> VOID Session::NotifyResult(ResultsData* pResultsData)
   |   |   |   |   |   |   |   |   |--> switch (pResultsData->type)
   |   |   |   |   |   |   |   |   |--> case CbType::Buffer:
   |   |   |   |   |   |   |   |   |--> HandleBufferCb(&pResultsData->cbPayload.buffer, pResultsData->pipelineIndex, pResultsData->pPrivData);
   |   |   |   |   |   |   |   |   |   |--> InjectResult(ResultType::BufferOK, &outBuffer, pPayload->sequenceId, pPrivData, pipelineIndex);
   |   |   |   |   |   |   |   |   |   |   |--> result = m_pThreadManager->PostJob(m_hJobFamilyHandle, NULL, &pData[0], FALSE, FALSE); //这里的线程cb是CHISession::ThreadJobCallback,为什么呢？
                                           file: vendor/qcom/proprietary/camx/src/core/chi/camxchisession.cpp
   |   |   |   |   |   |   |   |   |   |   |--> VOID* CHISession::ThreadJobCallback(VOID* pData)
   |   |   |   |   |   |   |   |   |   |   |   |--> result = pSession->ThreadJobExecute();
   |   |   |   |   |   |   |   |   |   |   |   |   |--> result = ProcessResults();
                                                   file: vendor/qcom/proprietary/camx/src/core/camxsession.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |--> CamxResult Session::ProcessResults()
   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> LightweightDoublyLinkedListNode* pNode = m_resultHolderList.Head();
   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> bufferReady = ProcessResultBuffers(pResultHolder, metadataReady, &numResults);  //针对每一个node都调用这个方法，感觉这里就是全部node处理buffer的一个过程，后期需要确认下
   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> pNode = m_resultHolderList.NextNode(pNode); //取出下一个node进行操作
   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> DispatchResults(&m_pCaptureResult[0], numResults);  //发送results
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> m_chiCallBacks.ChiProcessCaptureResult(&pCaptureResults[index], m_pPrivateCbData);  //这里的m_chiCallBacks的介绍见func_list_camx_chi ⑦
                                                           file: vendor/qcom/proprietary/chi-cdk/vendor/chioverride/default/chxadvancedcamerausecase.h
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |-->  static VOID      ProcessResultCb(...)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> static_cast<AdvancedCameraUsecase*>(pCbData->pUsecase)->ProcessResult(pResult, pPrivateCallbackData);
                                                               file: vendor/qcom/proprietary/chi-cdk/vendor/chioverride/default/chxadvancedcamerausecase.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> VOID AdvancedCameraUsecase::ProcessResult(...)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |-->  pFeature->ProcessResult(pResult, pPrivateCallbackData);    //如果是AdvancedFeatureEnabled,这里我们考虑的是预览，所以不是这分支
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> CameraUsecaseBase::SessionCbCaptureResult(pResult, pPrivateCallbackData);
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> pCameraUsecase->SessionProcessResult(pCaptureResult, pSessionPrivateData);
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> camera3_capture_result_t* pUsecaseResult     = GetCaptureResult(resultFrameIndex);
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> ChxUtils::PopulateChiToHALStreamBuffer(&pResult->pOutputBuffers[i], pResultBuffer);
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> ProcessAndReturnFinishedResults();
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> Usecase::ReturnFrameworkResult(&result, m_cameraId);
                                                                               file: vendor/qcom/proprietary/chi-cdk/vendor/chioverride/default/chxusecase.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> VOID Usecase::ReturnFrameworkResult(...)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> ExtensionModule::GetInstance()->ReturnFrameworkResult(pResult, cameraID);
                                                                                   file: vendor/qcom/proprietary/chi-cdk/vendor/chioverride/default/chxextensionmodule.cpp
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> VOID ExtensionModule::ReturnFrameworkResult(const camera3_capture_result_t* pResult, UINT32 cameraID)
   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |--> m_pHALOps->process_capture_result(m_logicalCameraInfo[cameraID].m_pCamera3Device, pResult); //这里就将result给到上面了

```

#### 2.3.4 Flush

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/2118860-cc06eef5b214bab3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、Qcom Hal3 Camx 重点

下面是具体项目上的工作者需要知道的camera相关的点,具体细节不赘述,这里记录下,方便记忆.

### 3.1 定制化

定制pipeline要走那些算法及节点配置在xml中,类似鱼如下路径
vendor/qcom/proprietary/chi-cdk/vendor/topology/qcom/sdm845/sdm845_usecase.xml
源码下可能因为共线有多个,要看清楚makefile中指定的是哪个.

bypass 是高通做的一套框架，目的在于减少不必要的拷贝。

### 3.2 性能perflock

高通的perflock调试,需要掌握,可以通过抓取systrace来看cpu相关的变化,也可以通过读取
/sys/devices/system/cpu/下的信息来调查cpu变化情况.

### 3.3 常见的node及作用

| Node | 作用 |
| :-: | :-- |
| BPS | Bayer Processing Segment，Bayer处理阶段。仅做snapshot的噪点降低和Bayer处理，不良像素、PDAF、LSC校正，绿色不平衡校正，黑色级别，通道增益，demosaic，Down scaler，HDR的合并与记录，Bayer混合降噪等 |
| IFE | Image front-end engine，图像前端引擎。仅做video/preview的Bayer处理，做些颜色校正，Down scaler，demosaic，统计3A数据等 |
| IPE | Image-processing engine，图像处理引擎。由NPS、PPS两部分组成，主要做些硬件降噪（MFNR、MFSR）、调整大小、噪声处理、颜色处理（色差校正、色度抑制）、细节增强（肤色增强） |
| JPEG | 打包jpeg |
| STATS | for 3A,ISP硬件给出的3A数据，用于后面的3A算法 |
IS :|图像稳定 (Image Stabilization)
RDI :|原始数据转储接口 (Raw Dump Interface)
RoI AF :|感兴趣区域 (AF Region of Interest)
SNoC 系统:| NoC (System NoC)
SOF：|start of frame
ANR：|application not response
MCC: |mutil camera control
LPM: |low power manager(低功耗下运行)
CTS/ITS ：|Android Camera Image Test Suite (ITS) is part of Android Compatibility Test Suite (CTS),Android相机图像测试套件（ITS）是Android兼容性测试套件（CTS）的一部分验证程序，包括验证图像内容的测试。从CTS 7.0_r8开始，CTS Verifier通过一体式摄像机ITS支持ITS测试自动化。继续支持手动测试，以确保涵盖所有Android设备尺寸。
HAF：|混合自动变焦
CRM: |camera request manager
Sensor CRA(主光线角）:|从镜头的传感器一侧，可以聚焦到像素上的光线的最大角度被定义为一个参数，称为主光角(CRA)。对于主光角的一般性定义是：此角度处的像素响应降低为零度角像素响应(此时，此像素是垂直于光线的)的80%,https://blog.csdn.net/weixin_39839293/article/details/82118991，lens CRA与sensor 不配会使sensor 的pixel 出现在光检测区域周围，使pixel 曝光不足，亮度不够，会使整个画面造成亮度不均匀的情况。还有可能造成chroma shading 或局部色偏。局部色偏比较严重，无法用算法补偿
Sensor HDR：|sensor在一幅图像里能够同时体现高光和阴影部分内容的能力
lens fov：|视场角,视场角与焦距的关系：一般情况下，视场角越大，焦距就越短.
IFE ：|Image Front End， Bayer processing for video/preview only， HDR/De-mosic， color correction ，scaler，也可以直接输出Raw到RDI
RDI : |Raw Dump Interface，直接从IFE吐出来用于capture的
STATS: |for 3A,ISP硬件给出的3A数据，用于后面的3A算法
PDPC：|PhaseDetection Pixel Correction，相位检测像素校正
ASD: |Auto scene detection，自动场景检测
CSIC:|Camera serial interface decoder,摄像机串行接口解码器
CAMIF: |Ideal Raw的第一个dump点
FD: |Face-based,基于人脸，也就是人脸识别
ICA：|图像校正和调整      是一个硬件单元，主要用于由镜头和运动引起的几何扭曲
LENR：|低/中频增强和降噪
TMC：|色调映射控制
CSID：|摄像机串行接口解码器模块
IFE：|图像前端
BPS：|Bayer处理段
IPE：|图像处理引擎
VPU：|视频处理单元
DPU：|显示处理单元
BPC：|坏像素校正
BCC：|坏群集校正
ABF：|自适应拜耳滤波器
GIC：|绿色不平衡校正
GTM：|全局色调映射
HNR：|混合降噪
ANR：|先进的降噪功能
TF：|时间过滤器
MFNR：|多帧降噪
LTM：|局部色调映射
CS：|色度抑制
ASF：|自适应空间滤波器
Upscaler：|升频器
GRA：|Grain Adder（纹理增加器？）？？
CPAS：|相机外围设备和支持
CAMIF：|摄像头接口？？？它是VFE（video front-end）硬件的第一部分，主要任务是同步sensor发送数据过程中涉及到的行、场同步信号。另外它还具有图像提取和图像镜像能力，CAMIF hardware使外部camera sensor能够通过一些简单的外部协议链接到用户单元。为camera提供了数据和时钟接口，但并没有提供控制接口，最具代表行的是用I2C做配置和状态接口。当然，也可以是其他的一次控制信号做一些静态控制。例如：睡眠唤醒模式控制。
NPS：|噪声处理部分
PPS：|后处理部分
MCTF：|运动补偿时间滤波 CAC、CCM、GLUT、2D LUT（二维查找表？）、CV（颜色转换）、CC（颜色校正）、SCE（肤色增强）、MCE（记忆色彩增强）：？？？
SIMO：|单输入多输出
Pedestal Correction：|基座校正
Down Scaler：|降低规模（尺寸）
Chroma Enhancement：|色度增强
Chroma Suppression：|色度抑制
PDAF：|相位检测自动对焦
LSC：|镜头阴影校正
PNR：|峰值降噪
ADRC：|自动动态范围压缩
Backlit scene：|背光场景
Garage scene：|车库场景
HNR：|降低亮度噪声，但保持纹理细节
LDC：|镜头畸变校正
EIS：|电子稳像
Multi pass spatial noise filtering：|多通空间噪声滤波
LNR：|镜头降噪
Invert gamma：|反转伽玛
hue, saturation, lightness：|色调，饱和度，亮度
Upscaler：|升频器
ACE：|高级色度增强
CPP：|相机后处理器（相当于新版的BPS、IPE）
BSP:|board support package，板级支持安装包？也就是“做出支持安装包，来实现手机上各个硬件的基本功能”。
CCT：|correlated color temperature，相关色温，具体不详；
chi-cdk：|Camera Hardward Interface 相机硬件接口；Camera Development Kit，相机开发包；
HFR：|High Frame Rate, min HFR=90, means>=90时，需要enable HFR高帧率，目前最高960，但是是利用插值算法计算得出的，非实际960

 

### 3.4 日志资料

Camx日志由属性控制,具体也是一套规则,首先获取权限：`adb root && adb remount`,如果remount报错：`failed: Read-only file system `,则执行`adb disable-verity && adb reboot` 解决，然后重新`adb root && adb remount`.

Camera user mode driver (UMD)  ——相机用户模式驱动

UMD的显示格式
```
CamX:  [<Verbosity Level>][<Group>]  <File>:<Line Number>  <Function Name> <Message>
```
例如：CamX    : [ INFO][HAL    ] camxhal3.cpp:1086 process_capture_request() frame_number 140

这里将log分为不同的level，不同的level下面有不同的group

如何打开指定level的指定group的log？其各level和group见相应的camxtypes.h

如何打开指定level?　　level直接通过Name设定；

group通过二进制的每一位来确定，每个group对应一个bit,置1表示打开

例如：logVerboseMask=0xFFFFFFFF       //打开所有group的verbose级别的log

             logWarningMask=0x82      //Warning级别打开第1个位和第6位表示的group，其实是Sensor和HAL的group

##### 平常测试开的log:           

如何实际进行打开log 　

在手机中的： /vendor/etc/camera/camxoverridesettings.txt　
```
例如:adb shell "echo logInfoMask=0xFFFFFFFF>> /vendor/etc/camera/camxoverridesettings.txt" 
```
##### chi log:
 
例如:CHX_LOG_ERROR(fmt, args)；
```
adb shell "echo overrideLogLevels=0x1f >> /vendor/etc/camera/camxoverridesettings.txt"
或：adb shell setprop vendor.debug.camera.overrideLogLevels 0x1F (camxsettings.xml中定义，不同的代码可能有区别)
```
#####平常开的log:

```
adb shell "echo logInfoMask=0x50080 >> /vendor/etc/camera/camxoverridesettings.txt" 　　hal/core/chi

adb shell setprop persist.vendor.camera.logVerboseMask 0xFFFFFFFF
adb shell setprop persist.vendor.camera.logEntryExitMask 0xFFFFFFFF
adb shell setprop persist.vendor.camera.logInfoMask 0xFFFFFFFF
adb shell setprop persist.vendor.camera.logWarningMask 0xFFFFFFFF
adb shell setprop persist.vendor.camera.logConfigMask 0xFFFFFFFF
adb shell setprop persist.vendor.camera.systemLogEnable TRUE
adb shell setprop persist.vendor.camera.logLogDRQMask 0xFFFFFFFF
```
##### camx log:
```
adb shell "echo overrideLogLevels=0xF >> /vendor/etc/camera/camxoverridesettings.txt" 
adb shell "echo logVerboseMask=0x1000 >> /vendor/etc/camera/camxoverridesettings.txt"
adb shell "echo logInfoMask=0xFFFFFFFF >> /vendor/etc/camera/camxoverridesettings.txt"
```
 
原文链接：https://blog.csdn.net/TaylorPotter/article/details/105630341



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

