```cmd
Traceback (most recent call last):
  File "D:\program\UGATIT-pytorch-app\tools\extract_video_frame.py", line 59, in <module>
    subprocess.call([
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 349, in call
    with Popen(*popenargs, **kwargs) as p:
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 951, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 1436, in _execute_child
    hp, ht, pid, tid = _winapi.CreateProcess(executable, args,
FileNotFoundError: [WinError 2] 系统找不到指定的文件。
```

```cmd
Traceback (most recent call last):
  File "D:\program\UGATIT-pytorch-app\tools\convert_frame_to_video.py", line 93, in <module>
    bit_rate = int(get_bit_rate(input_file))*4
  File "D:\program\UGATIT-pytorch-app\tools\convert_frame_to_video.py", line 49, in get_bit_rate
    json_str = subprocess.check_output(
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 424, in check_output
    return run(*popenargs, stdout=PIPE, timeout=timeout, check=True,
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 505, in run
    with Popen(*popenargs, **kwargs) as process:
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 951, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "D:\miniconda\envs\anime\lib\subprocess.py", line 1436, in _execute_child
    hp, ht, pid, tid = _winapi.CreateProcess(executable, args,
FileNotFoundError: [WinError 2] 系统找不到指定的文件。
```

