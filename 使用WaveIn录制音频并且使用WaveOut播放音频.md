使用WaveIn录制音频并且使用WaveOut播放音频
原创孟建行 发布于2018-06-14 13:22:34 阅读数 1306  收藏
展开

在Windows下开发音频的方式有多种，但是最简单，也是最灵活的就是Wave系列API，今天我们一起用WaveIn和WaveOut实现一个音频录制和音频播放器，具体界面如下：





录制步骤如下：

void CcbdDlg::OnBnClickedButtonStartRec()
{
	// TODO:  在此添加控件通知处理程序代码
	m_iHour = 0;
	m_iMinute = 0;
	m_iSec = 0;
 
	m_pDataBuff1 = NULL;
	m_pDataBuff2 = NULL;
	m_hWaveDev = NULL;
 
	m_dwFileSize = 0;
 
	CStringA strPathA;
 
	strPathA = m_strPath;
 
	m_fSave = fopen(strPathA.GetBuffer(strPathA.GetLength()), "ab+");
 
	if (!m_fSave)
	{
		AfxMessageBox(_T("文件打开失败!"));
		return;
	}
 
	WAVEFORMATEX waveCtx;
	waveCtx.nSamplesPerSec = 44100; /* sample rate */
	waveCtx.wBitsPerSample = 16; /* sample size */
	waveCtx.nChannels = 2; /* channels*/
	waveCtx.cbSize = 0; /* size of _extra_ info */
	waveCtx.wFormatTag = WAVE_FORMAT_PCM;
	waveCtx.nBlockAlign = (waveCtx.wBitsPerSample * waveCtx.nChannels) >> 3;
	waveCtx.nAvgBytesPerSec = waveCtx.nBlockAlign * waveCtx.nSamplesPerSec;
 
	MMRESULT iRet = waveInOpen(&m_hWaveDev, m_comboDevList.GetCurSel()/*WAVE_MAPPER*/, &waveCtx, (DWORD)(UINT)this->m_hWnd, NULL, CALLBACK_WINDOW);
 
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInOpen OK!\n");
	}
	else
	{
		AfxMessageBox(_T("设备打开失败!"));
		return;
	}
 
	//准备第一个缓冲
 
	m_pDataBuff1 = new BYTE[waveCtx.nAvgBytesPerSec];
	memset(m_pDataBuff1, 0, waveCtx.nAvgBytesPerSec);
	m_waveHdr1.lpData = (LPSTR)m_pDataBuff1;
	m_waveHdr1.dwBufferLength = waveCtx.nAvgBytesPerSec;
	m_waveHdr1.dwBytesRecorded = 0;
	m_waveHdr1.dwFlags = 0;
	iRet = waveInPrepareHeader(m_hWaveDev, &m_waveHdr1, sizeof(WAVEHDR));
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInPrepareHeader OK!\n");
	}
	else
	{
		if (m_pDataBuff1)
		{
			delete [] m_pDataBuff1;
			m_pDataBuff1 = NULL;
		}
		AfxMessageBox(_T("内存分配失败!"));
		return;
	}
	iRet = waveInAddBuffer(m_hWaveDev, &m_waveHdr1, sizeof(WAVEHDR));
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInAddBuffer OK!\n");
	}
	else
	{
		if (m_pDataBuff1)
		{
			delete [] m_pDataBuff1;
			m_pDataBuff1 = NULL;
		}
 
		AfxMessageBox(_T("录制准备失败!"));
		return;
	}
 
 
	//准备第二个缓冲
	m_pDataBuff2 = new BYTE[waveCtx.nAvgBytesPerSec];
	memset(m_pDataBuff2, 0, waveCtx.nAvgBytesPerSec);
	m_waveHdr2.lpData = (LPSTR)m_pDataBuff2;
	m_waveHdr2.dwBufferLength = waveCtx.nAvgBytesPerSec;
	m_waveHdr2.dwBytesRecorded = 0;
	m_waveHdr2.dwFlags = 0;
	iRet = waveInPrepareHeader(m_hWaveDev, &m_waveHdr2, sizeof(WAVEHDR));
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInPrepareHeader OK!\n");
	}
	else
	{
		if (m_pDataBuff1)
		{
			delete[] m_pDataBuff1;
			m_pDataBuff1 = NULL;
		}
		if (m_pDataBuff2)
		{
			delete [] m_pDataBuff2;
			m_pDataBuff2 = NULL;
		}
 
		AfxMessageBox(_T("准备录制声音失败!"));
		return;
	}
	iRet = waveInAddBuffer(m_hWaveDev, &m_waveHdr2, sizeof(WAVEHDR));
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInAddBuffer OK!\n");
	}
	else
	{
		if (m_pDataBuff1)
		{
			delete[] m_pDataBuff1;
			m_pDataBuff1 = NULL;
		}
 
		if (m_pDataBuff2)
		{
			delete [] m_pDataBuff2;
			m_pDataBuff2 = NULL;
		}
 
		AfxMessageBox(_T("缓冲添加失败!"));
		return;
	}
 
	iRet = waveInStart(m_hWaveDev);
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInStart OK!\n");
	}
	else
	{
		if (m_pDataBuff1)
		{
			delete [] m_pDataBuff1;
			m_pDataBuff1 = NULL;
		}
		if (m_pDataBuff2)
		{
			delete [] m_pDataBuff2;
			m_pDataBuff2 = NULL;
		}
		AfxMessageBox(_T("录制启动失败!"));
		return;
	}
 
	GetDlgItem(IDC_BUTTON_START_REC)->EnableWindow(FALSE);
	GetDlgItem(IDC_STATIC_REC_TIMER)->ShowWindow(SW_SHOW);
	GetDlgItem(IDC_STATIC_REC_TIMER)->SetWindowText(_T("00:00:00"));
	SetTimer(MM_AUDIO_RECORD_TIMER, 1000, NULL);
	SetTimer(SCREEN_RECORD_TIMER, 33, NULL);
}
上面的代码我们已经准备好了录制，当麦克风有音频信息传入的时候，我们会收到MM_WIM_DATA消息，在这里我们把收到的数据保存为文件，具体代码如下：

LRESULT CcbdDlg::WindowProc(UINT message, WPARAM wParam, LPARAM lParam)
{
	MMRESULT iRet = 0;
	switch (message)
	{
	case MM_WIM_OPEN:
		TRACE("Open!\n");
		break;
	case MM_WIM_CLOSE:
		TRACE("Close!\n");
		break;
	case MM_WIM_DATA:
 
		if (m_hWaveDev == NULL)
			break;
 
		LPWAVEHDR waveHdr = (LPWAVEHDR)lParam;
 
		if (!m_fSave)
		{
			TRACE("Error!\n");
			break;
		}
 
		BYTE bType = 1;//音频
 
		DWORD dwSize = waveHdr->dwBytesRecorded; //大小
 
		m_dwFileSize += dwSize;
 
		fwrite(&bType, 1, 1, m_fSave);
 
		fwrite(&dwSize, 1, 4, m_fSave);
 
		int nTotal = 0;
 
		while (nTotal < waveHdr->dwBytesRecorded)
		{
			int nTmp = fwrite(waveHdr->lpData, 1, waveHdr->dwBytesRecorded, m_fSave);
			nTotal += nTmp;
		}
 
		iRet = waveInAddBuffer(m_hWaveDev, waveHdr, sizeof(WAVEHDR));
 
		if (MMSYSERR_NOERROR != iRet)
		{
			if (m_pDataBuff1)
			{
				delete m_pDataBuff1;
				m_pDataBuff1 = NULL;
			}
			if (m_pDataBuff2)
			{
				delete m_pDataBuff2;
				m_pDataBuff2 = NULL;
			}
		}	
 
		break;
	}
 
	return CDialog::WindowProc(message, wParam, lParam);
}
当用户点击结束录制的时候，我们需要做一些善后的工作，具体如下：

void CcbdDlg::OnBnClickedButtonEndRec()
{
	// TODO:  在此添加控件通知处理程序代码
 
	waveInUnprepareHeader(m_hWaveDev, &m_waveHdr1, sizeof(WAVEHDR));
	waveInUnprepareHeader(m_hWaveDev, &m_waveHdr2, sizeof(WAVEHDR));
	MMRESULT iRet = waveInStop(m_hWaveDev);
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInStop OK!\n");
	}
	else
	{
		TRACE("no,waveInStop Wrong!\n");
	}
	iRet = waveInReset(m_hWaveDev);
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInReset OK!\n");
	}
	else
	{
		TRACE("no,waveInReset Wrong!\n");
	}
	iRet = waveInClose(m_hWaveDev);
	if (MMSYSERR_NOERROR == iRet)
	{
		TRACE("yes,waveInClose OK!\n");
	}
	else
	{
		TRACE("no,waveInClose Wrong!\n");
	}
	m_hWaveDev = NULL;
	if (m_pDataBuff1)
	{
		delete m_pDataBuff1;
		m_pDataBuff1 = NULL;
	}
	if (m_pDataBuff2)
	{
		delete m_pDataBuff2;
		m_pDataBuff2 = NULL;
	}
	GetDlgItem(IDC_BUTTON_START_REC)->EnableWindow(TRUE);
	GetDlgItem(IDC_STATIC_REC_TIMER)->ShowWindow(SW_HIDE);
	KillTimer(MM_AUDIO_RECORD_TIMER);
 
	if (m_fSave)
	{
		fwrite(&m_dwFileSize, 1, 4, m_fSave);
	}
 
	if (m_fSave)
	{
		fclose(m_fSave);
		return;
	}
}
上面我们完成了录制，下面我们一起看一下播放。

播放器的具体开发步骤如下：

当用户点击播放的时候，我们首先需要读取音频文件，用音频文件的数据初始化两个缓冲区，然后提交给声卡，让其播放，具体代码如下：

void CcbdplayerDlg::OnBnClickedButtonPlay()
{
	// TODO:  在此添加控件通知处理程序代码
 
	m_bStartPlay = true;
 
	CStringA strTmpA;
	strTmpA = m_strPath;
	m_fTmp = fopen(strTmpA.GetBuffer(strTmpA.GetLength()), "rb+");
	if (!m_fTmp)
	{
		AfxMessageBox(_T("文件打开失败!"));
		return;
	}
 
	fseek(m_fTmp, -4, SEEK_END);
 
	fread(&m_dwFileSize, 1, 4, m_fTmp);
 
	fseek(m_fTmp, 0, SEEK_SET);
 
	int nTmp = m_dwFileSize;
	
	m_PlayProgress.SetRange32(0, nTmp);
 
	m_PlayProgress.SetPos(0);
 
	MMRESULT mmresult = 0;
	mmresult = waveOutOpen(&m_hWaveOut, WAVE_MAPPER, &m_WaveFormat, (DWORD)m_hWnd, (DWORD_PTR)0, CALLBACK_WINDOW);
	if (mmresult != MMSYSERR_NOERROR)
	{
		AfxMessageBox(_T("打开设备失败!"));
		return;
	}
	
	if (!m_pDataBuff1)
	{
		AfxMessageBox(_T("内存分配失败!"));
		return;
	}
 
	int nReadSize = 0;
 
	BYTE bType = 0;
 
	DWORD dwSize = 0;
 
	fread(&bType, 1, 1, m_fTmp);
 
	fread(&dwSize, 1, 4, m_fTmp);
 
	memset(m_pDataBuff1, 0, m_WaveFormat.nAvgBytesPerSec);
 
	nReadSize = fread(m_pDataBuff1, 1, dwSize, m_fTmp);
 
	m_WaveHead1.lpData = (LPSTR)m_pDataBuff1;
	m_WaveHead1.dwBufferLength = nReadSize;
	m_WaveHead1.dwBytesRecorded = 0;
	m_WaveHead1.dwUser = 0;
	m_WaveHead1.dwFlags = 0;
	m_WaveHead1.dwLoops = 1;
	m_WaveHead1.lpNext = NULL;
	m_WaveHead1.reserved = 0;
 
	mmresult = waveOutPrepareHeader(m_hWaveOut, &m_WaveHead1, sizeof(WAVEHDR));
	if (mmresult != MMSYSERR_NOERROR)
	{
		if (m_pDataBuff1)
			delete[] m_pDataBuff1;
		if (m_pDataBuff2)
			delete[] m_pDataBuff2;
 
		AfxMessageBox(_T("准备失败!"));
 
		return;
	}
 
	memset(m_pDataBuff2, 0, m_WaveFormat.nAvgBytesPerSec);
 
	fread(&bType, 1, 1, m_fTmp);
 
	fread(&dwSize, 1, 4, m_fTmp);
 
	nReadSize = fread(m_pDataBuff2, 1, dwSize, m_fTmp);
 
	if (!m_pDataBuff2)
	{
		if (m_pDataBuff1)
			delete[] m_pDataBuff1;
 
		AfxMessageBox(_T("内存分配失败!"));
		return;
	}
 
	m_WaveHead2.lpData = (LPSTR)m_pDataBuff2;
	m_WaveHead2.dwBufferLength = nReadSize;
	m_WaveHead2.dwBytesRecorded = 0;
	m_WaveHead2.dwUser = 0;
	m_WaveHead2.dwFlags = 0;
	m_WaveHead2.dwLoops = 1;
	m_WaveHead2.lpNext = NULL;
	m_WaveHead2.reserved = 0;	
 
	mmresult = waveOutPrepareHeader(m_hWaveOut, &m_WaveHead2, sizeof(WAVEHDR));
 
	if (mmresult != MMSYSERR_NOERROR)
	{
		if (m_pDataBuff1)
			delete[] m_pDataBuff1;
		if (m_pDataBuff2)
			delete[] m_pDataBuff2;
 
		AfxMessageBox(_T("准备失败!"));
 
		return;
	}
	
	waveOutWrite(m_hWaveOut, &m_WaveHead1, sizeof(WAVEHDR));
 
	waveOutWrite(m_hWaveOut, &m_WaveHead2, sizeof(WAVEHDR));
	
	GetDlgItem(IDC_BUTTON_PLAY)->EnableWindow(FALSE);
 
	GetDlgItem(IDC_BUTTON_STOP_PLAY)->EnableWindow(TRUE);
}
当声卡播放完我们提交给它的数据后，它会发送通知WOM_DONE给我们，我们需要做的是继续从文件读取音频，然后再次提交给声卡，具体代码如下：

LRESULT CcbdplayerDlg::WindowProc(UINT message, WPARAM wParam, LPARAM lParam)
{
	MMRESULT iRet = 0;
 
	switch (message)
	{
	case WOM_OPEN:
		break;
	case WOM_DONE:
	{
		if (m_bStartPlay == false)
			break;
 
		LPWAVEHDR pWaveHeader = (LPWAVEHDR)lParam;
 
		if (pWaveHeader)
		{
			int nPos = m_PlayProgress.GetPos();
 
			if (nPos >= m_dwFileSize)
			{
				break;
			}
 
			nPos += pWaveHeader->dwBufferLength;
			m_PlayProgress.SetPos(nPos);
 
			waveOutUnprepareHeader(m_hWaveOut, pWaveHeader, sizeof(WAVEHDR));
 
			int nReadSize = 0;
 
			BYTE bType = 0;
 
			DWORD dwSize = 0;
 
			fread(&bType, 1, 1, m_fTmp);			
 
			if (feof(m_fTmp))
				break;
 
			if (bType != 1)
				break;
 
			fread(&dwSize, 1, 4, m_fTmp);
 
			if (feof(m_fTmp))
				break;
 
			memset(pWaveHeader->lpData, 0, m_WaveFormat.nAvgBytesPerSec);
 
			nReadSize = fread(pWaveHeader->lpData, 1, dwSize, m_fTmp);
 
			if (feof(m_fTmp))
				break;
 
			pWaveHeader->dwBufferLength = nReadSize;
 
			MMRESULT mmresult = waveOutPrepareHeader(m_hWaveOut, pWaveHeader, sizeof(WAVEHDR));
 
			if (mmresult != MMSYSERR_NOERROR)
			{
				AfxMessageBox(_T("准备失败!"));
				break;
			}
 
			waveOutWrite(m_hWaveOut, pWaveHeader, sizeof(WAVEHDR));
		}
 
	}
		break;
	}
 
	return CDialog::WindowProc(message, wParam, lParam);
}
当用户点击停止播放的时候，我们需要做的是释放资源，具体如下：

void CcbdplayerDlg::OnBnClickedButtonStopPlay()
{
	// TODO:  在此添加控件通知处理程序代码
 
	waveOutReset(m_hWaveOut);
 
	waveOutUnprepareHeader(m_hWaveOut, &m_WaveHead1, sizeof(WAVEHDR));
 
	waveOutUnprepareHeader(m_hWaveOut, &m_WaveHead2, sizeof(WAVEHDR));
 
	waveOutClose(m_hWaveOut);
 
	if (m_fTmp)
	{
		fclose(m_fTmp);
		m_fTmp = NULL;
	}
 
	GetDlgItem(IDC_BUTTON_PLAY)->EnableWindow(TRUE);
 
	GetDlgItem(IDC_BUTTON_STOP_PLAY)->EnableWindow(FALSE);
 
	m_bStartPlay = false;
 
	return;
}
需要代码的小伙伴，可以从下面的地址下载：

https://download.csdn.net/download/u011711997/10478662
————————————————
版权声明：本文为CSDN博主「孟建行」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u011711997/article/details/80691325
