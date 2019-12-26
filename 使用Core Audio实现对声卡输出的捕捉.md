使用Core Audio实现对声卡输出的捕捉
涉及的接口有：

IMMDeviceEnumerator
IMMDevice
IAudioClient
IAudioCaptureClient
主要过程：

创建多媒体设备枚举器(IMMDeviceEnumerator)
通过多媒体设备枚举器获取声卡接口(IMMDevice)
通过声卡接口获取声卡客户端接口(IAudioClient)
通过声卡客户端接口(IAudioClient)可获取声卡输出的音频参数、初始化声卡、获取声卡输出缓冲区的大小、开启/停止对声卡输出的采集
通过声卡采集客户端接口(IAudioCaptureClient)可获取采集的声卡输出数据，并对内部缓冲区进行控制
通过声卡输出客户端接口(IAudioRenderClient)可将采集的声音数据写入声音输出设备的缓存单元，输出设备可以通过IAudioClient::GetService函数获取 IAudioRenderClient 接口。
     

代码示例：

   

//首先枚举你的音频设备，你可以在这个时候获取到你机器上所有可用的设备，并指定你需要用到的那个设备

hr = CoCreateInstance(

                    CLSID_MMDeviceEnumerator, NULL,
                    CLSCTX_ALL, IID_IMMDeviceEnumerator,
                    (void**)&pEnumerator);
EXIT_ON_ERROR(hr)

   

hr = pEnumerator->GetDefaultAudioEndpoint(

                    eCapture, eConsole, &pDevice);
EXIT_ON_ERROR(hr)

   

   

//创建一个管理对象，通过它可以获取到你需要的一切数据
hr = pDevice->Activate(
                    IID_IAudioClient, CLSCTX_ALL,
                    NULL, (void**)&pAudioClient);
EXIT_ON_ERROR(hr)
   

hr = pAudioClient->GetMixFormat(&pwfx);
EXIT_ON_ERROR(hr)
   

//初始化管理对象，在这里，你可以指定它的最大缓冲区长度，这个很重要，应用程序控制数据块的大小以及延时长短都靠这里的初始化，具体参数大家看看文档解释
hr = pAudioClient->Initialize(
                         AUDCLNT_SHAREMODE_SHARED,
                         0,
                         hnsRequestedDuration,
                         0,
                         pwfx,
                         NULL);
EXIT_ON_ERROR(hr)
   

//这个buffersize，指的是缓冲区最多可以存放多少帧的数据量
hr = pAudioClient->GetBufferSize(&bufferFrameCount);
EXIT_ON_ERROR(hr)
 

//创建采集管理接口，这个接口很简单，没什么重要的东西
hr = pAudioClient->GetService(
                         IID_IAudioCaptureClient,
                         (void**)&pCaptureClient);
EXIT_ON_ERROR(hr)
 

// Notify the audio sink which format to use.
hr = pMySink->SetFormat(pwfx);
EXIT_ON_ERROR(hr)
   

// Calculate the actual duration of the allocated buffer.
hnsActualDuration = (double)REFTIMES_PER_SEC *
                     bufferFrameCount / pwfx->nSamplesPerSec;
   

hr = pAudioClient->Start();  // Start recording.

EXIT_ON_ERROR(hr)
   

   

// Each loop fills about half of the shared buffer.
while (bDone == FALSE)
{
    //让程序暂停运行一段时间，缓冲区里在这段时间会被填充数据                        
        Sleep(hnsActualDuration/REFTIMES_PER_MILLISEC/2);
   

        hr = pCaptureClient->GetNextPacketSize(&packetLength);
        EXIT_ON_ERROR(hr)
   

    while (packetLength != 0)
        {
        //锁定缓冲区，获取数据
        hr = pCaptureClient->GetBuffer(
                                   &pData,
                                   &numFramesAvailable,
                                   &flags, NULL, NULL);
                EXIT_ON_ERROR(hr)
   

      if (flags & AUDCLNT_BUFFERFLAGS_SILENT)
           {
                pData = NULL;  
           }
     

            hr = pMySink->CopyData(
                              pData, numFramesAvailable, &bDone);
            EXIT_ON_ERROR(hr)
     

            hr = pCaptureClient->ReleaseBuffer(numFramesAvailable);
            EXIT_ON_ERROR(hr)
     

            hr = pCaptureClient->GetNextPacketSize(&packetLength);
            EXIT_ON_ERROR(hr)
        }
    }
   

hr = pAudioClient->Stop(); 
EXIT_ON_ERROR(hr)


查看录制的音频可以用 audacity 这个软件
