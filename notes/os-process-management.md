# OS Process Management

## Links

- http://msdn.microsoft.com/en-us/library/windows/desktop/ms741565(v=vs.85).aspx
- http://stackoverflow.com/questions/670891/is-there-a-way-for-multiple-processes-to-share-a-listening-socket
- http://stackoverflow.com/questions/23434842/python-how-to-kill-child-processes-when-parent-dies
- http://www.evans.io/posts/killing-child-processes-on-parent-exit-prctl/
- http://stackoverflow.com/questions/1173342/terminate-a-process-tree-c-for-windows
- http://stackoverflow.com/questions/985281/what-is-the-closest-thing-windows-has-to-fork
- http://bugs.java.com/bugdatabase/view_bug.do?bug_id=4770092
- http://svn.smedbergs.us/python-processes/trunk/killableprocess.py
- http://serverfault.com/questions/293632/how-do-i-start-a-process-in-suspended-state-under-linux
- http://www.cs.huji.ac.il/~feit/papers/ClustBaseTR.pdf

## Code Snippets

killableprocess.py:
```
# killableprocess - subprocesses which can be reliably killed
#
# Parts of this module are copied from the subprocess.py file contained
# in the Python distribution.
#
# Copyright (c) 2003-2004 by Peter Astrand <astrand@lysator.liu.se>
#
# Additions and modifications written by Benjamin Smedberg
# <benjamin@smedbergs.us> are Copyright (c) 2006 by the Mozilla Foundation
# <http://www.mozilla.org/>
#
# By obtaining, using, and/or copying this software and/or its
# associated documentation, you agree that you have read, understood,
# and will comply with the following terms and conditions:
#
# Permission to use, copy, modify, and distribute this software and
# its associated documentation for any purpose and without fee is
# hereby granted, provided that the above copyright notice appears in
# all copies, and that both that copyright notice and this permission
# notice appear in supporting documentation, and that the name of the
# author not be used in advertising or publicity pertaining to
# distribution of the software without specific, written prior
# permission.
#
# THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE,
# INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS.
# IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, INDIRECT OR
# CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS
# OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT,
# NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION
# WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

r"""killableprocess - Subprocesses which can be reliably killed

This module is a subclass of the builtin "subprocess" module. It allows
processes that launch subprocesses to be reliably killed on Windows (via the Popen.kill() method.

It also adds a timeout argument to Wait() for a limited period of time before
forcefully killing the process.

Note: On Windows, this module requires Windows 2000 or higher (no support for
Windows 95, 98, or NT 4.0). It also requires ctypes, which is bundled with
Python 2.5+ or available from http://python.net/crew/theller/ctypes/
"""

import subprocess
import sys
import os
import time
import types

try:
    from subprocess import CalledProcessError
except ImportError:
    # Python 2.4 doesn't implement CalledProcessError
    class CalledProcessError(Exception):
        """This exception is raised when a process run by check_call() returns
        a non-zero exit status. The exit status will be stored in the
        returncode attribute."""
        def __init__(self, returncode, cmd):
            self.returncode = returncode
            self.cmd = cmd
        def __str__(self):
            return "Command '%s' returned non-zero exit status %d" % (self.cmd, self.returncode)

mswindows = (sys.platform == "win32")

if mswindows:
    import winprocess
else:
    import signal

def call(*args, **kwargs):
    waitargs = {}
    if "timeout" in kwargs:
        waitargs["timeout"] = kwargs.pop("timeout")

    return Popen(*args, **kwargs).wait(**waitargs)

def check_call(*args, **kwargs):
    """Call a program with an optional timeout. If the program has a non-zero
    exit status, raises a CalledProcessError."""

    retcode = call(*args, **kwargs)
    if retcode:
        cmd = kwargs.get("args")
        if cmd is None:
            cmd = args[0]
        raise CalledProcessError(retcode, cmd)

if not mswindows:
    def DoNothing(*args):
        pass

class Popen(subprocess.Popen):
    if not mswindows:
        # Override __init__ to set a preexec_fn
        def __init__(self, *args, **kwargs):
            if len(args) >= 7:
                raise Exception("Arguments preexec_fn and after must be passed by keyword.")

            real_preexec_fn = kwargs.pop("preexec_fn", None)
            def setpgid_preexec_fn():
                os.setpgid(0, 0)
                if real_preexec_fn:
                    apply(real_preexec_fn)

            kwargs['preexec_fn'] = setpgid_preexec_fn

            subprocess.Popen.__init__(self, *args, **kwargs)

    if mswindows:
        def _execute_child(self, args, executable, preexec_fn, close_fds,
                           cwd, env, universal_newlines, startupinfo,
                           creationflags, shell,
                           p2cread, p2cwrite,
                           c2pread, c2pwrite,
                           errread, errwrite):
            if not isinstance(args, types.StringTypes):
                args = subprocess.list2cmdline(args)

            if startupinfo is None:
                startupinfo = winprocess.STARTUPINFO()

            if None not in (p2cread, c2pwrite, errwrite):
                startupinfo.dwFlags |= winprocess.STARTF_USESTDHANDLES
                
                startupinfo.hStdInput = int(p2cread)
                startupinfo.hStdOutput = int(c2pwrite)
                startupinfo.hStdError = int(errwrite)
            if shell:
                startupinfo.dwFlags |= winprocess.STARTF_USESHOWWINDOW
                startupinfo.wShowWindow = winprocess.SW_HIDE
                comspec = os.environ.get("COMSPEC", "cmd.exe")
                args = comspec + " /c " + args

            # We create a new job for this process, so that we can kill
            # the process and any sub-processes 
            self._job = winprocess.CreateJobObject()

            creationflags |= winprocess.CREATE_SUSPENDED
            creationflags |= winprocess.CREATE_UNICODE_ENVIRONMENT

            hp, ht, pid, tid = winprocess.CreateProcess(
                executable, args,
                None, None, # No special security
                1, # Must inherit handles!
                creationflags,
                winprocess.EnvironmentBlock(env),
                cwd, startupinfo)
            
            self._child_created = True
            self._handle = hp
            self._thread = ht
            self.pid = pid

            winprocess.AssignProcessToJobObject(self._job, hp)
            winprocess.ResumeThread(ht)

            if p2cread is not None:
                p2cread.Close()
            if c2pwrite is not None:
                c2pwrite.Close()
            if errwrite is not None:
                errwrite.Close()

    def kill(self, group=True):
        """Kill the process. If group=True, all sub-processes will also be killed."""
        if mswindows:
            if group:
                winprocess.TerminateJobObject(self._job, 127)
            else:
                winprocess.TerminateProcess(self._handle, 127)
            self.returncode = 127    
        else:
            if group:
                os.killpg(self.pid, signal.SIGKILL)
            else:
                os.kill(self.pid, signal.SIGKILL)
            self.returncode = -9

    def wait(self, timeout=-1, group=True):
        """Wait for the process to terminate. Returns returncode attribute.
        If timeout seconds are reached and the process has not terminated,
        it will be forcefully killed. If timeout is -1, wait will not
        time out."""

        if self.returncode is not None:
            return self.returncode

        if mswindows:
            if timeout != -1:
                timeout = timeout * 1000
            rc = winprocess.WaitForSingleObject(self._handle, timeout)
            if rc == winprocess.WAIT_TIMEOUT:
                self.kill(group)
            else:
                self.returncode = winprocess.GetExitCodeProcess(self._handle)
        else:
            if timeout == -1:
                subprocess.Popen.wait(self)
                return self.returncode
            
            starttime = time.time()

            # Make sure there is a signal handler for SIGCHLD installed
            oldsignal = signal.signal(signal.SIGCHLD, DoNothing)

            while time.time() < starttime + timeout - 0.01:
                pid, sts = os.waitpid(self.pid, os.WNOHANG)
                if pid != 0:
                    self._handle_exitstatus(sts)
                    signal.signal(signal.SIGCHLD, oldsignal)
                    return self.returncode
                
                # time.sleep is interrupted by signals (good!)
                newtimeout = timeout - time.time() + starttime
                time.sleep(newtimeout)

            self.kill(group)
            signal.signal(signal.SIGCHLD, oldsignal)
            subprocess.Popen.wait(self)

        return self.returncode
```

winproces.py:
```
# A module to expose various thread/process/job related structures and
# methods from kernel32
#
# The MIT License
#
# Copyright (c) 2006 the Mozilla Foundation <http://www.mozilla.org>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.

from ctypes import c_void_p, POINTER, sizeof, Structure, windll, WinError, WINFUNCTYPE
from ctypes.wintypes import BOOL, BYTE, DWORD, HANDLE, LPCWSTR, LPWSTR, UINT, WORD

LPVOID = c_void_p
LPBYTE = POINTER(BYTE)
LPDWORD = POINTER(DWORD)

def ErrCheckBool(result, func, args):
    """errcheck function for Windows functions that return a BOOL True
    on success"""
    if not result:
        raise WinError()
    return args

# CloseHandle()

CloseHandleProto = WINFUNCTYPE(BOOL, HANDLE)
CloseHandle = CloseHandleProto(("CloseHandle", windll.kernel32))
CloseHandle.errcheck = ErrCheckBool

# AutoHANDLE

class AutoHANDLE(HANDLE):
    """Subclass of HANDLE which will call CloseHandle() on deletion."""
    def Close(self):
        if self.value:
            CloseHandle(self)
            self.value = 0
    
    def __del__(self):
        self.Close()

    def __int__(self):
        return self.value

def ErrCheckHandle(result, func, args):
    """errcheck function for Windows functions that return a HANDLE."""
    if not result:
        raise WinError()
    return AutoHANDLE(result)

# PROCESS_INFORMATION structure

class PROCESS_INFORMATION(Structure):
    _fields_ = [("hProcess", HANDLE),
                ("hThread", HANDLE),
                ("dwProcessID", DWORD),
                ("dwThreadID", DWORD)]

    def __init__(self):
        Structure.__init__(self)
        
        self.cb = sizeof(self)

LPPROCESS_INFORMATION = POINTER(PROCESS_INFORMATION)

# STARTUPINFO structure

class STARTUPINFO(Structure):
    _fields_ = [("cb", DWORD),
                ("lpReserved", LPWSTR),
                ("lpDesktop", LPWSTR),
                ("lpTitle", LPWSTR),
                ("dwX", DWORD),
                ("dwY", DWORD),
                ("dwXSize", DWORD),
                ("dwYSize", DWORD),
                ("dwXCountChars", DWORD),
                ("dwYCountChars", DWORD),
                ("dwFillAttribute", DWORD),
                ("dwFlags", DWORD),
                ("wShowWindow", WORD),
                ("cbReserved2", WORD),
                ("lpReserved2", LPBYTE),
                ("hStdInput", HANDLE),
                ("hStdOutput", HANDLE),
                ("hStdError", HANDLE)
                ]
LPSTARTUPINFO = POINTER(STARTUPINFO)

STARTF_USESHOWWINDOW    = 0x01
STARTF_USESIZE          = 0x02
STARTF_USEPOSITION      = 0x04
STARTF_USECOUNTCHARS    = 0x08
STARTF_USEFILLATTRIBUTE = 0x10
STARTF_RUNFULLSCREEN    = 0x20
STARTF_FORCEONFEEDBACK  = 0x40
STARTF_FORCEOFFFEEDBACK = 0x80
STARTF_USESTDHANDLES    = 0x100

# EnvironmentBlock

class EnvironmentBlock:
    """An object which can be passed as the lpEnv parameter of CreateProcess.
    It is initialized with a dictionary."""

    def __init__(self, dict):
        if not dict:
            self._as_parameter_ = None
        else:
            values = ["%s=%s" % (key, value)
                      for (key, value) in dict.iteritems()]
            values.append("")
            self._as_parameter_ = LPCWSTR("\0".join(values))
        
# CreateProcess()

CreateProcessProto = WINFUNCTYPE(BOOL,                  # Return type
                                 LPCWSTR,               # lpApplicationName
                                 LPWSTR,                # lpCommandLine
                                 LPVOID,                # lpProcessAttributes
                                 LPVOID,                # lpThreadAttributes
                                 BOOL,                  # bInheritHandles
                                 DWORD,                 # dwCreationFlags
                                 LPVOID,                # lpEnvironment
                                 LPCWSTR,               # lpCurrentDirectory
                                 LPSTARTUPINFO,         # lpStartupInfo
                                 LPPROCESS_INFORMATION  # lpProcessInformation
                                 )

CreateProcessFlags = ((1, "lpApplicationName", None),
                      (1, "lpCommandLine"),
                      (1, "lpProcessAttributes", None),
                      (1, "lpThreadAttributes", None),
                      (1, "bInheritHandles", True),
                      (1, "dwCreationFlags", 0),
                      (1, "lpEnvironment", None),
                      (1, "lpCurrentDirectory", None),
                      (1, "lpStartupInfo"),
                      (2, "lpProcessInformation"))

def ErrCheckCreateProcess(result, func, args):
    ErrCheckBool(result, func, args)
    # return a tuple (hProcess, hThread, dwProcessID, dwThreadID)
    pi = args[9]
    return AutoHANDLE(pi.hProcess), AutoHANDLE(pi.hThread), pi.dwProcessID, pi.dwThreadID

CreateProcess = CreateProcessProto(("CreateProcessW", windll.kernel32),
                                   CreateProcessFlags)
CreateProcess.errcheck = ErrCheckCreateProcess

CREATE_BREAKAWAY_FROM_JOB = 0x01000000
CREATE_DEFAULT_ERROR_MODE = 0x04000000
CREATE_NEW_CONSOLE = 0x00000010
CREATE_NEW_PROCESS_GROUP = 0x00000200
CREATE_NO_WINDOW = 0x08000000
CREATE_SUSPENDED = 0x00000004
CREATE_UNICODE_ENVIRONMENT = 0x00000400
DEBUG_ONLY_THIS_PROCESS = 0x00000002
DEBUG_PROCESS = 0x00000001
DETACHED_PROCESS = 0x00000008

# CreateJobObject()

CreateJobObjectProto = WINFUNCTYPE(HANDLE,             # Return type
                                   LPVOID,             # lpJobAttributes
                                   LPCWSTR             # lpName
                                   )

CreateJobObjectFlags = ((1, "lpJobAttributes", None),
                        (1, "lpName", None))

CreateJobObject = CreateJobObjectProto(("CreateJobObjectW", windll.kernel32),
                                       CreateJobObjectFlags)
CreateJobObject.errcheck = ErrCheckHandle

# AssignProcessToJobObject()

AssignProcessToJobObjectProto = WINFUNCTYPE(BOOL,      # Return type
                                            HANDLE,    # hJob
                                            HANDLE     # hProcess
                                            )
AssignProcessToJobObjectFlags = ((1, "hJob"),
                                 (1, "hProcess"))
AssignProcessToJobObject = AssignProcessToJobObjectProto(
    ("AssignProcessToJobObject", windll.kernel32),
    AssignProcessToJobObjectFlags)
AssignProcessToJobObject.errcheck = ErrCheckBool

# ResumeThread()

def ErrCheckResumeThread(result, func, args):
    if result == -1:
        raise WinError()

    return args

ResumeThreadProto = WINFUNCTYPE(DWORD,      # Return type
                                HANDLE      # hThread
                                )
ResumeThreadFlags = ((1, "hThread"),)
ResumeThread = ResumeThreadProto(("ResumeThread", windll.kernel32),
                                 ResumeThreadFlags)
ResumeThread.errcheck = ErrCheckResumeThread

# TerminateJobObject()

TerminateJobObjectProto = WINFUNCTYPE(BOOL,   # Return type
                                      HANDLE, # hJob
                                      UINT    # uExitCode
                                      )
TerminateJobObjectFlags = ((1, "hJob"),
                           (1, "uExitCode", 127))
TerminateJobObject = TerminateJobObjectProto(
    ("TerminateJobObject", windll.kernel32),
    TerminateJobObjectFlags)
TerminateJobObject.errcheck = ErrCheckBool

# WaitForSingleObject()

WaitForSingleObjectProto = WINFUNCTYPE(DWORD,  # Return type
                                       HANDLE, # hHandle
                                       DWORD,  # dwMilliseconds
                                       )
WaitForSingleObjectFlags = ((1, "hHandle"),
                            (1, "dwMilliseconds", -1))
WaitForSingleObject = WaitForSingleObjectProto(
    ("WaitForSingleObject", windll.kernel32),
    WaitForSingleObjectFlags)

INFINITE = -1
WAIT_TIMEOUT = 0x0102
WAIT_OBJECT_0 = 0x0
WAIT_ABANDONED = 0x0080

# GetExitCodeProcess()

GetExitCodeProcessProto = WINFUNCTYPE(BOOL,    # Return type
                                      HANDLE,  # hProcess
                                      LPDWORD, # lpExitCode
                                      )
GetExitCodeProcessFlags = ((1, "hProcess"),
                           (2, "lpExitCode"))
GetExitCodeProcess = GetExitCodeProcessProto(
    ("GetExitCodeProcess", windll.kernel32),
    GetExitCodeProcessFlags)
GetExitCodeProcess.errcheck = ErrCheckBool
```