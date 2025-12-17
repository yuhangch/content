---
id: arD
title: Some Java Tips (Brief Notes)
pubDate: 2019-12-16T08:32:01.000Z
isDraft: true
tags:
  - java
categories:
  - 笔记
---

From the beginning of my first year of graduate school up to the end of the second year, I’ve handled several projects. This is a collection of some issues encountered in a Java backend process and their solutions.

### Calling a C++ dynamic library via JNA

First define the model interface as follows:

```java
package ch.yuhang.XModel;

import com.sun.jna.Library;
import com.sun.jna.Native;
import com.sun.jna.Pointer;

public interface XModel extends Library {
    XModel Instance = (XModel) Native.load(PATH_TO_MODEL_DLL, OSPModel.class);
    Pointer RunModel(String arg1, String arg2, String arg3);
}
```

`PATH_TO_MODEL_DLL` points to the dll file; the suffix can be included or omitted. The call looks like this:

```java
OSPModel ospModel = OSPModel.Instance;
Pointer string = XModel.RunModel(arg1, arg2 , arg3 );
```

Return values are supported, and the command-line output from the dll itself will also take effect.

### Java command-line call blocking and exception handling

In the project, because it needed to integrate with other functionality, in addition to using a dynamic library we also directly invoked Python scripts and exe programs. Early on, there was an issue where the command ran successfully the first time but failed to run normally the second time. The reason was that printed output was causing a block. Using the following approach resolved the problem:

```java
package ch.yuhang.XModel;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class Cmd {
    public static int run(String command,String desc) {
        int re = 10;
        try {
            System.out.println(command);
            Process process = Runtime.getRuntime().exec(command);
            BufferedReader in = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line = null;
            while ((line = in.readLine()) != null) {
                System.out.println(line);
            }
            in.close();
            re = process.waitFor();
            if (re == 0) {
                System.out.printf("%s --> success\n",desc);
            } else {
                System.out.printf("%s --> failed\n",desc);
            }
            return re;

        } catch (Exception e) {
            e.printStackTrace();
            return re;
        }

    }
}
```

The two parameters are: a string containing the command and its arguments, and a description of the command (used for printing execution status). A return value of `0` from `re = process.waitFor();` indicates that the task completed successfully; otherwise an exception occurred.

Referenced from [shendeguang](https://blog.csdn.net/shendeguang/article/details/17854079)’s blog.

### Formatting strings with specified argument positions

When formatting output, sometimes you have repeated parameters. You can use `num$` to specify argument positions, as shown below:

```java
String format = "Hi ,%s %s,you first name is %1$s";
String message = String.format(format,"yuhang","chen");
```

### Outputting a Java Json object as an INI configuration file

This involves some legacy functionality. A previous project used .NET, and some calls required configuration items to be written in INI format, while the frontend returned data in Json format. This raised the problem of how to convert between them. Manually writing everything was a bit troublesome, so I found the following method. The basic idea is to iterate over the fields of the Object, then format and output the `key` and `value`.

```java
package ch.yuhang.XModel;

import com.google.gson.Gson;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public static boolean writeINI(Object json,String uuid) throws NoSuchMethodException, InvocationTargetException {
        StringBuffer path = new StringBuffer(Path.EVAL_SHEET_TEMP);
        path.append(uuid).append(".ini");

        FileWriter w = null;
        try {
            File file = new File(path.toString());
            file.createNewFile();
            // true means not overwriting the original content, but appending to the end of the file.
            // If you want to overwrite the original content, just omit this parameter.
            //System.out.println(path.toString());
            w = new FileWriter(path.toString());

            Gson gson = new Gson();
            String a = gson.toJson(json);
            EvalSheet e = gson.fromJson(a, EvalSheet.class);
            Field[] fields = e.getClass().getDeclaredFields();

            for (Field f:
                    fields) {
                String name = f.getName();
                f.setAccessible(true);
                w.write(String.format("[%s]\n",name));

                try {

                    Object l =  f.get(e);
                    Field[] fsTmp = l.getClass().getDeclaredFields();
                    if (fsTmp.length == 0) {
                        continue;
                    }
                    for (Field f2:
                            fsTmp) {
                        f2.setAccessible(true);
                        String nameTmp = f2.getName();
                        //String typeTmp = f2.getGenericType().toString();
                        Method m = l.getClass().getMethod("get" + nameTmp);
                        String value = (String) m.invoke(l);    // call getter method to get property value
                        if (value != null)
                        {
                            w.write(String.format("%s = %s\n",nameTmp,value));
                        }
                    }

                } catch (IllegalAccessException er ) {
                }
            }
        } catch (IOException ex) {
            ex.printStackTrace();
            return false;
        } finally {
            try {
                w.flush();
                w.close();

            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        return true;

    }
```