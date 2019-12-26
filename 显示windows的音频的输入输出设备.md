

#include "stdafx.h"

/************************音频的输入输出设备**************************/

#include <Mmdeviceapi.h>
#include <Functiondiscoverykeys_devpkey.h>
#include <locale.h>

#define EXIT_ON_ERROR(hres)  \
              if (FAILED(hres)) { goto Exit; }
#define SAFE_RELEASE(punk)  \
              if ((punk) != NULL)  \
                             { (punk)->Release(); (punk) = NULL; }

const CLSID CLSID_MMDeviceEnumerator = __uuidof(MMDeviceEnumerator);
const IID IID_IMMDeviceEnumerator = __uuidof(IMMDeviceEnumerator);

void PrintEndpointNames()
{
    HRESULT hr = S_OK;
    IMMDeviceEnumerator *pEnumerator = NULL;
    IMMDeviceCollection *pRenderCollection = NULL;
    IMMDeviceCollection *pCaptureCollection = NULL;
    IMMDevice *pEndpoint = NULL;
    IPropertyStore *pProps = NULL;
    LPWSTR pwszID = NULL;

    hr = CoCreateInstance(
        CLSID_MMDeviceEnumerator, NULL,
        CLSCTX_ALL, IID_IMMDeviceEnumerator,
        (void**)&pEnumerator);
    EXIT_ON_ERROR(hr)

        //switch eRender or eCapture
        hr = pEnumerator->EnumAudioEndpoints(
        eCapture, DEVICE_STATE_ACTIVE,
        &pRenderCollection);

    EXIT_ON_ERROR(hr)

        UINT  count;
    hr = pRenderCollection->GetCount(&count);

    EXIT_ON_ERROR(hr)

        if (count == 0)
        {
            printf("No endpoints found.\n");
        }

    // Each loop prints the name of an endpoint device.
    for (ULONG i = 0; i < count; i++)
    {
        // Get pointer to endpoint number i.
        hr = pRenderCollection->Item(i, &pEndpoint);
        EXIT_ON_ERROR(hr)

            // Get the endpoint ID string.
            hr = pEndpoint->GetId(&pwszID);
        EXIT_ON_ERROR(hr)

            hr = pEndpoint->OpenPropertyStore(
            STGM_READ, &pProps);
        EXIT_ON_ERROR(hr)

            PROPVARIANT varName;
        // Initialize container for property value.
        PropVariantInit(&varName);

        // Get the endpoint's friendly-name property.
        hr = pProps->GetValue(
            PKEY_Device_FriendlyName, &varName);
        EXIT_ON_ERROR(hr)


            setlocale(LC_ALL, "chs");//中文宽字符输出
        // Print endpoint friendly name and endpoint ID.
        wprintf(L"Endpoint %d: \"%s\" ( %s )\n",
            i, varName.pwszVal, pwszID);

        CoTaskMemFree(pwszID);
        pwszID = NULL;
        PropVariantClear(&varName);
        SAFE_RELEASE(pProps)
            SAFE_RELEASE(pEndpoint)
    }
    SAFE_RELEASE(pEnumerator)
        SAFE_RELEASE(pRenderCollection)
        return;

Exit:
    printf("Error!\n");
    CoTaskMemFree(pwszID);
    SAFE_RELEASE(pEnumerator)
        SAFE_RELEASE(pRenderCollection)
        SAFE_RELEASE(pEndpoint)
        SAFE_RELEASE(pProps)
}

/************************音频的输入输出设备**************************/



int _tmain(int argc, _TCHAR* argv[])
{
    CoInitializeEx(NULL, COINIT_MULTITHREADED);

    PrintEndpointNames();

    getchar();

    return 0;
}
