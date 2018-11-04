#### IDA7.0_SP
- Fixed when multiply input method the IDA pro7.0 on mojave, ida will crash in non-english input method.
- Fixed shortcuts do not work in non-english input method. Eg: F2, tab, ctrl+enter etc.
- Added load bundle for open dialog (The official Qt does not support this feature, but this is suitable for ida
)
```
Replace the "libqcocoa.dylib" to 
/Applications/IDA Pro 7.0/ida.app/Contents/PlugIns/platforms/libqcocoa.dylib
```



#### Binary file checksum:

```
md5 libqcocoa.dylib
MD5 (libqcocoa.dylib) = 9c8fa1ec2d16bc18e326f362918b0cb9

shasum libqcocoa.dylib
23d94e8dae902515f1587e4bda8292e536c8e25a  libqcocoa.dylib
```





#### Detail:
#### Added load bundle for open dialog
![](./images/load_bundle_open_dlg.png)

#### When multiply input method the IDA pro7.0 on mojave, ida will crash in non-english input method.

![](./images/ida7.0_crash.png)
------

##### Solution: 
##### ida7.0 using Qt5.6

##### **crash stack:**

```
Exception Type:        EXC_BAD_ACCESS (SIGSEGV)
Exception Codes:       KERN_INVALID_ADDRESS at 0x00003eadde8f1958
Exception Note:        EXC_CORPSE_NOTIFY

Termination Signal:    Segmentation fault: 11
Termination Reason:    Namespace SIGNAL, Code 0xb
Terminating Process:   exc handler [13886]  

Application Specific Information:
objc_msgSend() selector name: length

Thread 0 Crashed:: Dispatch queue: com.apple.main-thread
0   libobjc.A.dylib               	0x00007fff686faa1d objc_msgSend + 29
1   org.qt-project.QtCore         	0x0000000101903924 QT::QCFString::toQString(__CFString const*) + 52
2   libqcocoa.dylib               	0x0000000103508f67 0x1034c0000 + 298855
```

Cause:
``` c++
    TISInputSourceRef source = TISCopyCurrentKeyboardInputSource();
    CFArrayRef languages = (CFArrayRef) TISGetInputSourceProperty(source, kTISPropertyInputSourceLanguages);
    if (CFArrayGetCount(languages) > 0) {
        CFStringRef langRef = (CFStringRef)CFArrayGetValueAtIndex(languages, 0);
        QString name = QCFString::toQString(langRef);
        QLocale locale(name);
        if (m_locale != locale) {
            m_locale = locale;
            emitLocaleChanged();
        }
        CFRelease(langRef);
    }
```

**langRef** dangling pointer

#### Fixed:
``` c++
#import <Foundation/Foundation.h>

void QCocoaInputContext::updateLocale()
{
    /* https://bugreports.qt.io/browse/QTBUG-48772

    TISInputSourceRef source = TISCopyCurrentKeyboardInputSource();
    CFArrayRef languages = (CFArrayRef) TISGetInputSourceProperty(source, kTISPropertyInputSourceLanguages);
    if (CFArrayGetCount(languages) > 0) {
        CFStringRef langRef = (CFStringRef)CFArrayGetValueAtIndex(languages, 0);
        QString name = QCFString::toQString(langRef);
        QLocale locale(name);
        if (m_locale != locale) {
            m_locale = locale;
            emitLocaleChanged();
        }
        CFRelease(langRef);
    }
    
    */

    QString name = QString::fromNSString([[NSLocale currentLocale] objectForKey:NSLocaleIdentifier]);
    QLocale locale(name);
    if (m_locale != locale) {
        m_locale = locale;
        emitLocaleChanged();
    }
} 
```

#### Recompile Qt5.6
**Download** 
```
https://download.developer.apple.com/Developer_Tools/Xcode_7.3.1/Xcode_7.3.1.dmg
```

**Switch**
```
sudo xcode-select -switch /Applications/Xcode7.app/Contents/Developer
```

**Compile argument**

```
sh configure "-nomake" "tests" "-qtnamespace" "QT" "-confirm-license" "-accessibility" "-opensource" "-force-debug-info" "-platform" "macx-g++" "-debug-and-release" "-fontconfig" "-qt-freetype" "-qt-libpng" "-qt-sql-sqlite" "-prefix" "Qt/5.6.0-x64"
```
FAQ

Xcode not set up properly. You may need to confirm the license agreement by running /usr/bin/xcodebuild.


```
Replace "/usr/bin/xcrun -find xcrun" 
to "/usr/bin/xcrun -find xcodebuild"  
```

