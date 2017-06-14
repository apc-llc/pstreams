## PStreams with clone() instead of fork()

This is a fork of Jonathan Wakely's PStream software, which has been modified to use clone() in places of fork() calls.

The fork() call is known to potentially cause problems in MPI applications. Particularly in our case, fork() crashes on Cray machine:

```
#4  0x00002aaab2ea0034 in _int_free () from /lib64/libc.so.6
#5  0x00002aaab363404e in GNI_CqDestroy () from /opt/cray/ugni/default/lib64/libugni.so.0
#6  0x00002aaab36403c2 in GNII_DlaFree () from /opt/cray/ugni/default/lib64/libugni.so.0
#7  0x00002aaab36302ab in cdm_destroy.part.1 () from /opt/cray/ugni/default/lib64/libugni.so.0
#8  0x00002aaab3630890 in GNII_CdmFork () from /opt/cray/ugni/default/lib64/libugni.so.0
#9  0x00002aaab2ede33f in fork () from /lib64/libc.so.6
#10 0x00002aaab76ae8c1 in redi::basic_pstreambuf<char, std::char_traits<char> >::fork (this=this@entry=0x7fffffff4ba8, mode=mode@entry=std::_S_app) at ./pstreams/pstream.h:1351
```

The reason for this behavior is likely a failing `atfork` hook triggered somewhere from within the uGNI library (Cray's Gemini interconnect support). In order to overcome this issue, we basically replace fork() calls with clone(), which is not hooked.

