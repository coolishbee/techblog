## 유니티 2017 버전에서 useAndroidX 사용하기

안드로이드에서 참조하는 라이브러리 메서드가 65,536개를 초과하면 Mulidex 문제가 생긴다.

유니티 안드로이드빌드도 마찬가지로 적용된다.



그래서 유니티2019에서는 공식적으로 gradle.properties 를 지원해주고 있고 그안에서

안드로이드에서 처리하듯이

```
android.useAndroidX=true
```

만 넣어주면 해결이 된다.



하지만 유니티 2017 에서는 gradle.properties 를 지원하지 않는 것 같다.

이런 경우 해결방법을 제안해본다.



mainTemplate.gradle 파일내에서 추가

```
buildscript {
	repositories {        
		google()
		jcenter()		
	}

	dependencies {
            classpath 'com.android.tools.build:gradle:3.4.0'
	}
}
...
}

//added
([rootProject] + (rootProject.subprojects as List)).each {
	ext {
		it.setProperty("android.useAndroidX", true)
		it.setProperty("android.enableJetifier", true)
	}
}
```



아참 2018버전도 동일하게 사용해도 되며, 다른 방법도 있다.

```
public class AndroidPostBuildProcessor : IPostGenerateGradleAndroidProject
{
    public int callbackOrder
    {
        get
        {
            return 999;
        }
    }

    void IPostGenerateGradleAndroidProject.OnPostGenerateGradleAndroidProject(string path)
    {
        Debug.Log("Bulid path : " + path);
        string gradlePropertiesFile = path + "/gradle.properties";
        if (File.Exists(gradlePropertiesFile))
        {
            File.Delete(gradlePropertiesFile);
        }
        StreamWriter writer = File.CreateText(gradlePropertiesFile);
        writer.WriteLine("org.gradle.jvmargs=-Xmx4096M");
        writer.WriteLine("android.useAndroidX=true");
        writer.WriteLine("android.enableJetifier=true");
        writer.Flush();
        writer.Close();

    }
}
```



> 참고자료
>
> https://stackoverflow.com/questions/54186051/is-there-a-way-to-change-the-gradle-properties-file-in-unity

