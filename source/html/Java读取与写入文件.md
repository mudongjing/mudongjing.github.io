**本文使用的版本为**Java 11

由于电脑中保存了大量的书籍，使用了everything检索出所有文档信息后，将对应的路径信息保存到一个txt文件中。
为了从中提取出文件名，需要剔除路径信息。这里顺便记录一下java的文件操作。

```python
import java.io.File;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.nio.file.Path;
import java.util.List;
import java.util.ArrayList;
import java.io.IOException;
import java.lang.Exception;
import java.util.Set;
import java.util.HashSet;
import java.nio.charset.StandardCharsets;
import java.nio.file.StandardOpenOption;
public class shuming{
    public static void main(String[] args) throws Exception {
        String filename=args[0];
		//在命令行中赋予参数，
		//指定当前目录下对应的文件名
        
		File pwd=new File("");
		//对应java文件所处路径
        
		String file=pwd.getCanonicalPath()+
		"\\"+filename;
		//getCanonicalPath()获取对应的路径名
        
		List<String> lines=fileReader(file);
		//自定义的读取文件的方法，且按行读取
        
		String newfile=pwd.getCanonicalPath()
		+"\\"+"onlybook_name.txt";
		//创建一个新的文件路径，
		//用于之后存储新的内容
        
		create_new_file(newfile);
		//自定义创建文件的方法
        
		List<String> newlines=new ArrayList<String>();
        int len=0;
        String line;
        while(len<lines.size() 
			  && ( line=lines.get(len++))!=null){
           
			String newline=line.
			substring(line.lastIndexOf("\\")+1,
					  line.length());
			//提取文件名
           
			newlines.add(newline);
	    }
		
	    Files.write(Paths.get(newfile), 
					newlines, StandardCharsets.UTF_8,
					StandardOpenOption.CREATE);
		//将nnewlines保存的文件名按行保存到新文件中
	}
    private static List<String> 
	fileReader(String file_name)
	throws IOException{
        Path path=Paths.get(file_name);
        
		return Files.readAllLines(path);
		//按行读取全部内容
    }

    private static void 
	create_new_file(String newfile)  
	throws Exception{
        File f=new File(newfile);
	   
		f.createNewFile();
		//createNewFile()为bool函数，
		//当不存在指定文件时，便创建，并返回true
    }
}
```