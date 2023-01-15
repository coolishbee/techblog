# Unity 에서 iOS Capability 활성화

Unity 2019.3 부터 사용법이 변경된 듯하다.

인앱결제, 푸쉬, 애플로그인 활성화 시키기



CapabilitySettings.cs

```
#if UNITY_IOS
using UnityEngine;
using UnityEditor;
using UnityEditor.Callbacks;
using UnityEditor.iOS.Xcode;
using System.IO;

namespace Game.SDK.Editor
{
    public class CapabilitySettings
    {
        [PostProcessBuildAttribute(1)]
        public static void OnPostprocessBuild(BuildTarget target, string pathToBuiltProject)
        {
            if (target != BuildTarget.iOS)
            {
                return;
            }
            
            string projectPath = PBXProject.GetPBXProjectPath(pathToBuiltProject);
            var project = new PBXProject();
            project.ReadFromFile(projectPath);
            string targetGuid = project.GetUnityMainTargetGuid();

            project.AddFrameworkToProject(targetGuid, "StoreKit.framework", false);
            File.WriteAllText(projectPath, project.WriteToString());

            var manager = new ProjectCapabilityManager(projectPath, "Entitlements.entitlements", null, project.GetUnityMainTargetGuid());
            manager.AddInAppPurchase();
            manager.AddPushNotifications(true);
            manager.AddSignInWithApple();
            manager.WriteToFile();            
        }
    }

}
#endif
```



> 출처
>
> https://forum.unity.com/threads/projectcapabilitymanager-addinapppurchase-not-working.940893/