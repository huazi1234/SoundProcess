
利用waveIn完成简单录音（录成wav）
原创玄岳 发布于2016-01-21 20:56:33 阅读数 6051  收藏
展开

设置采集音频格式
WAVEFORMATEX waveform; //采集音频的格式，结构体
waveform.wFormatTag = WAVE_FORMAT_PCM;//声音格式为PCM
waveform.nSamplesPerSec = 8000;//采样率，16000次/秒
waveform.wBitsPerSample = 16;//采样比特，16bits/次
waveform.nChannels = 1;//采样声道数，2声道
waveform.nAvgBytesPerSec = 16000;//每秒的数据率，就是每秒能采集多少字节的数据
waveform.nBlockAlign = 2;//一个块的大小，采样bit的字节数乘以声道数
waveform.cbSize = 0;//一般为0

提前准备好的录音数据缓存
m_pRecoderBuf = new unsigned char[20 * 1024];
m_recoderNum = 20;
m_recoderBufLen = 20 * 1024;
m_pWaveHdr	= new WAVEHDR[20];
memset(m_pRecoderBuf, 0, 20 * 1024);
memset(m_pWaveHdr, 0, sizeof(WAVEHDR) * 20);

打开录音设备
MMRESULT mmres = waveInOpen(&m_hWaveIn, WAVE_MAPPER, &waveform, (DWORD_PTR)WaveInProc, (DWORD_PTR)this, CALLBACK_FUNCTION );
	
if(mmres != MMSYSERR_NOERROR)
{
	// failed, try again.
	return;
}
WAVE_MAPPER表示系统会自己寻找合适的录音设备。
准备数据空间
for (int i = 0; i < m_recoderNum; ++i)
{
	m_pWaveHdr[i].lpData = (char*)m_pRecoderBuf + i * 1024;
	m_pWaveHdr[i].dwBufferLength = 1024;
	mmres = waveInPrepareHeader(m_hWaveIn, &m_pWaveHdr[i], sizeof(WAVEHDR));
	mmres = waveInAddBuffer(m_hWaveIn, &m_pWaveHdr[i], sizeof(WAVEHDR));
}


开始录音并打开写文件
mmres = waveInStart(m_hWaveIn);
 
SYSTEMTIME systemtime;
GetLocalTime(&systemtime);
char filename[256];
sprintf_s(filename, 256, "%d-%d-%d_%d_%d_%d.wav", systemtime.wYear, systemtime.wMonth, systemtime.wDay, systemtime.wHour, systemtime.wMinute, systemtime.wSecond);
m_pWavFile = wav_write_open(filename, 8000, 16, 1);

要提前定义好数据接收回调函数，因为我们用的是CALLBACK_FUNCTION
static BOOL CALLBACK WaveInProc(HWAVEOUT hwo, UINT uMsg, DWORD dwInstance, DWORD dwParam1, DWORD dwParam2);
BOOL   WaveInProcImpl(HWAVEOUT hwo, UINT uMsg, DWORD dwParam1, DWORD dwParam2);
函数里的处理是：
BOOL CAudioFileConvertDlg::WaveInProc(HWAVEOUT hwo, UINT uMsg, DWORD dwInstance, DWORD dwParam1, DWORD dwParam2)
{
	CAudioFileConvertDlg* pPlayer = (CAudioFileConvertDlg*)dwInstance;
 
	return pPlayer->WaveInProcImpl(hwo, uMsg, dwParam1, dwParam2);
}
 
BOOL CAudioFileConvertDlg::WaveInProcImpl(HWAVEOUT hwo, UINT uMsg, DWORD dwParam1, DWORD dwParam2)
{
	// 忽略打开和关闭设备时的处理
	if(uMsg == WIM_DATA)
	{
		LPWAVEHDR pHdr = (LPWAVEHDR) dwParam1;
 
		MMRESULT mmres = waveInUnprepareHeader (m_hWaveIn, pHdr, sizeof(WAVEHDR));
		//处理数据 
		if (NULL != m_pWavFile)
		{
			wav_write_data(m_pWavFile, (unsigned char *)(pHdr->lpData), pHdr->dwBytesRecorded);
		}
		//重新准备数据
		mmres = waveInPrepareHeader(m_hWaveIn, pHdr, sizeof(WAVEHDR));
		mmres = waveInAddBuffer(m_hWaveIn, pHdr, sizeof(WAVEHDR));
	}
 
	return TRUE;
}

关闭录音并关闭wav写文件
MMRESULT mmres = waveInStop(m_hWaveIn);
mmres = waveInClose(m_hWaveIn);
wav_write_close(m_pWavFile);
if (NULL != m_pRecoderBuf)
{
	delete []m_pRecoderBuf;
	m_pRecoderBuf = NULL;
}
 
if (NULL != m_pWaveHdr)
{
	delete []m_pWaveHdr;
	m_pWaveHdr = NULL;
}
简单在此记录下备忘
————————————————
版权声明：本文为CSDN博主「玄岳」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zyx4843/article/details/50557558
