[TOC]



# 파일 입출력

## 기본 파일 정보와 디렉토리 정보 다루기

- File/FileInfo, Directory/DirectoryInfo는 System.IO 에서 제공하는 기본 클래스들이다.

  - Info가 붙고 안붙냐의 차이는 정적 메소드로 사용하는지 (File/Directory) 

    혹은 인스턴스를 생성하는지(FileInfo/DirectoryInfo)의 차이이다.

    ```c#
    // File의 경우
    FileStream fs = File.Create("a.dat");
    
    // FileInfo의 경우
    FileInfo file = new FileInfo("a.dat");
    FileStream fs = file.Create();
    
    
    // Directory의 경우
    DirectoryInfo dir = Directory.CreateDirectory("a");
    
    // DirectoryInfo의 경우
    DirectoryInfo dir = new DirectoryInfo("a");
    dir.Create();
    ```

    



### directory, file 정보 얻기

```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace IO
{
    class Program
    {
 
        static void Main(string[] args)
        {
            string directory;
           
            if (args.Length < 1) // 매개변수가 없으면 현재 위치
            {
                directory = ".";
            }
            else // 매개변수가 없으면 입력한 위치
            {
                directory = args[0];
            }

            Console.WriteLine($"{directory} directory info");
            Console.WriteLine("- Directories :");

            var directories = (from dir in Directory.GetDirectories(directory) // 하위목록 조회
                               let info = new DirectoryInfo(dir)
                               select new
                               {
                                   Name = info.Name,
                                   Attribute = info.Attributes
                               }).ToList();

            foreach (var d in directories)
            {
                Console.WriteLine($"{d.Name} : {d.Attribute}");
            }

            Console.WriteLine("- Files :");
            var files = (from file in Directory.GetFiles(directory) // 하위 파일 목록 조회
                         let info = new FileInfo(file)
                         // let은 LINQ에서 var이라고 생각하면 된다.
                         select new
                         {
                             Name = info.Name,
                             FileSize = info.Length,
                             Attributes = info.Attributes
                         });

            foreach (var f in files)
            {
                Console.WriteLine($"{f.Name} : {f.FileSize}, {f.Attributes}");
            }

        }
    }
}

```

```
. directory info
- Directories :
- Files :
helloworld.txt : 25, Archive
IO.exe : 10752, Archive
IO.exe.config : 189, Archive
IO.pdb : 13824, Archive
IO.vshost.exe : 22696, Archive
IO.vshost.exe.config : 189, Archive
IO.vshost.exe.manifest : 490, Archive
```



### directory / file 생성

```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace IO1_Create
{
    class Program
    {
        static void OnWrongPathType(string type)
        {
            Console.WriteLine($"{type} is wrong type");
            return;
        }

        static void Main(string[] args)
        {
            if (args.Length == 0)
            {
                Console.WriteLine(
                    "Usage : Program.exe <Path> [Type:File/Directory]");
                return;
            }

            string path = args[0];
            string type = "File"; // 기본적으로 File이라 설정
            if (args.Length > 1)
                type = args[1]; // Directory가 들어오는지 확인


            if (File.Exists(path) || Directory.Exists(path))
            {
                if (type == "File")
                    File.SetLastWriteTime(path, DateTime.Now);
                else if (type == "Directory")
                    Directory.SetLastWriteTime(path, DateTime.Now);
                else
                {
                    OnWrongPathType(path);
                    return;
                }
                Console.WriteLine($"Updated {path}, {type}");
            }
            else
            {
                if (type == "File")
                    File.Create(path).Close();
                else if (type == "Directory")
                    Directory.CreateDirectory(path);
                else
                {
                    OnWrongPathType(path);
                    return;
                }

                Console.WriteLine($"Created {path} {type}");
            }

        }
    }
}

```

```
// cmd

>IO1_Create.exe
result : Usage : IO1_Create.exe <Path> [Type:/File/Directory]

>IO1_Create.exe a.dat
Created a.dat File

>>IO1_Create.exe MyFolder Directory
Created MyFolder Directory
```







