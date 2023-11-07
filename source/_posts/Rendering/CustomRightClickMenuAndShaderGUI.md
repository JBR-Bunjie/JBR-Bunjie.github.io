---
title: Custom Right-Click Menu And Shader GUI排列
date: 2023-4-10 16:32:30
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Custom Right-Click Menu And Shader GUI

> All the operations in this project should be done in a folder named \"Editor\", no matter where this folder is though.

## What do we want to do?

we want to set up a custom file template in the right-click menu, so we can quickly create the files that we need.

To archive this, we need to:

- Set up the file template
- Create C# Scripts which could be embedded into the editor to archive the file creating logic

In this project, I will take the "URP Unlit Shader" as an example. And we will learn how to custom the shader ui later, too.

## Set up a template for shader

I will put a simple example to explain:

```cpp
Shader "Custom URP Shader/#NAME#" {
    Properties {}
    SubShader {
        Tags {"RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "Queue" = "Geometry"}
        Pass {
            Name "ForwardLit"
            Tags {"LightMode" = "UniversalForward"}

            HLSLPROGRAM
        	#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #pragma vertex vert
            #pragma fragment frag

            struct appdata {...};
            struct v2f {...};

            v2f vert(appdata v) {...}

            half4 frag(v2f i): SV_Target {...}
            ENDHLSL
        }
    }
    CustomEditor "ReArchiving.Editor.CreateURPShader"
}
```

There are three important parts:

- `Shader "Custom URP Shader/#NAME#"` in the first line, the `#NAME#` part will be replaced by the script with the file's name.
- `CustomEditor "ReArchiving.Editor.CreateURPShader"`, we use the `CustomEditor` command to archive the Custom UI of a single shader in the Inspector Tab. See: [Unity - Manual: ShaderLab: assigning a custom editor (unity3d.com)](https://docs.unity3d.com/Manual/SL-CustomEditor.html)
- And the rest part is the entity, we can leave the template code here.

## Custom Shader And its Tab

After defining the template, we need to define the control script!

Here is the code:

```csharp
// -----------------------------------------
// Base class named `CreateCustomItemInMenu`
// -----------------------------------------
using UnityEngine;
using UnityEditor;
using System.IO;
using System.Text;
using UnityEditor.ProjectWindowCallback;
using System.Text.RegularExpressions;

public class CreateCustomItemInMenu {
    public static string GetSelectedPathOrFallback() {
        string path = "Assets";

        foreach (Object obj in Selection.GetFiltered(typeof(Object), SelectionMode.Assets)) {
            path = AssetDatabase.GetAssetPath(obj);
            if (!string.IsNullOrEmpty(path) && File.Exists(path)) {
                path = Path.GetDirectoryName(path);
                break;
            }
        }

        return path;
    }
}


class EndAction : EndNameEditAction {
    public override void Action(int instanceId, string pathName, string resourceFile) {
        Object o = CreateScriptAssetFromTemplate(pathName, resourceFile);
        ProjectWindowUtil.ShowCreatedAsset(o);
    }

    private static Object CreateScriptAssetFromTemplate(string pathName, string resourceFile) {
        string fullPath = Path.GetFullPath(pathName);
        StreamReader streamReader = new StreamReader(resourceFile);
        string text = streamReader.ReadToEnd(); // read the template info
        streamReader.Close();

        string fileNameWithoutExtension = Path.GetFileNameWithoutExtension(pathName);
        text = Regex.Replace(text, "#NAME#", fileNameWithoutExtension); // replace `#NAME#` in template and replace to filename

        // write the changed resources into the original file
        bool encoderShouldEmitUTF8Identifier = true;
        bool throwOnInvalidBytes = false;
        UTF8Encoding encoding = new UTF8Encoding(encoderShouldEmitUTF8Identifier, throwOnInvalidBytes);

        bool append = false;
        StreamWriter streamWriter = new StreamWriter(fullPath, append, encoding);

        streamWriter.Write(text);
        streamWriter.Close();
        AssetDatabase.ImportAsset(pathName);

        return AssetDatabase.LoadAssetAtPath(pathName, typeof(Object));
    }
}

// -----------------------------------------
// Base class named `CreateURPShader`
// -----------------------------------------
using UnityEditor;
using UnityEngine;

public class CreateURPShader : CreateCustomItemInMenu {
    private const string TemplatePath = "Assets/CustomShaderGUI/Editor/Template/URPShader.shader";

    [MenuItem("Assets/Create/Shader/URP Shader")]
    public static void CreateFileFromTemplate() {
        ProjectWindowUtil.StartNameEditingIfProjectWindowExists(
            0,
            ScriptableObject.CreateInstance<EndAction>(),
            GetSelectedPathOrFallback() + "/URPShader.shader",
            null,
            TemplatePath
        );
    }
}
```

It's important to divide the function into parts so that we can reuse and debug easier.

And now you can see the template in the right-click menu:

![image-20230409172030993](../../images\Dev\Unity\Archives\CustomRightClickMenuAndShaderGUI\001.png)

## Define the Shader GUI

We can directly use the built-in properties of unity to custom the GUI, see:

> [Shader 面板上常用的一些内置枚举 UI - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/93194054) > [【Unity Shader】自定义材质面板的小技巧*unity shader bool*妈妈说女孩子要自立自强的博客-CSDN 博客](https://blog.csdn.net/candycat1992/article/details/51417965)

But here we choose to use scripts to custom the shader UI which is more powerful and flexible.

Here is the basic framework I suggested:

### Custom Shader GUI Though Scripts

#### Setup namespace:

As for the start, we need to derive from an abstract class `ShaderGUI`.
[Unity - Scripting API: ShaderGUI (unity3d.com)](https://docs.unity3d.com/ScriptReference/ShaderGUI.html)

> Derive from this class for controlling how shader properties should be presented.
>
> For a shader to use this custom GUI, use the `CustomEditor` property in the shader. Note that `CustomEditor` can also be used for classes deriving from `MaterialEditor` (search for: `Custom Material Editors`). Note: Only the ShaderGUI approach works with Substance materials this is therefore the recommended approach to custom gui for shaders.

```c#
namespace ReArchiving.Editor {
    public class SimpleShaderGUI : ShaderGUI {
        ...
    }
}
```

#### Declare variables

In this section, we have three things to do:

- declare all the GUIContent which we want to show aside. Suggest defining this section with the name `GUIDescriptions`.
- declare all the strings which we have defined inside the target shader. We can define more strings than properties in the target shader itself because we have to do more operations before using these variables and we can exclude the properties that do not exist in the shader easily at that time. Suggest defining this section with the name `InsideMaterialProperties`.
- declare all the status variables. We use them to decide should corresponding variables be displayed.

Here is the example code of this section:

```c#
        #region DataArea
        private struct GUIDescriptions {
            // Folder
            public static readonly GUIContent SurfaceOptionFold = new GUIContent("Surface Option Fold");

            // Material Properties
            public static readonly GUIContent Test = new GUIContent("Test");
            public static readonly GUIContent MainTex = new GUIContent("Main Texture");
            public static readonly GUIContent MainColor = new GUIContent("Main Color");
        }

        private struct InsideMaterialProperties {
            public static readonly string Test = "_Test";
            public static readonly string MainTex = "_MainTex";
            public static readonly string MainColor = "_MainColor";
        }

        // After `Styles` and `MporpertyNames` were defined, we can then announce the variables for checking:
        // As you can see, we will use two data types here: one is the `bool` and another one is the `MaterialProperty`
		// This is because we use the `bool` type to store the Foldout UI which is independent of Properties that are defined in Material
		// And relatively, the `MaterialProperties` will store the Material related to Properties.
        // There is an exception though, that is the `EditorPreKey`, we use that to

        // `EditorPreKey`, we use this to store the foldout status(is this foldout opened or not)
        private const string EditorPreKey = "ReArchiving:ShaderGUI:";

        // Foldout status
        private bool m_SurfaceOptionsFoldout;

        // Properties
        private MaterialProperty m_WorkflowModeProp;
        private MaterialProperty m_Test;
        private MaterialProperty m_MainTex;
        private MaterialProperty m_MainColor;
        #endregion
```

#### Override the `OnGUI` function:

Although we haven't defined the other functions we need to use, we define the override `OnGUI` function here to help us clear the things we need to do.
[Unity - Scripting API: ShaderGUI.OnGUI (unity3d.com)](https://docs.unity3d.com/ScriptReference/ShaderGUI.OnGUI.html)

So, what should we do here?

We need to decide which property, GUIConent and type we want to show up in the Inspector tab. So we need to **check the Usability of the properties** and **set them up**. And that's what we want to do.

And, before we start coding, we'd better understand the `OnGUI` function. This function has two parameters: `MaterialEditor` and `MaterialProperty`

- `MaterialEditor`: The MaterialEditor that is calling this OnGUI (the 'owner'). [Unity - Scripting API: MaterialEditor (unity3d.com)](https://docs.unity3d.com/ScriptReference/MaterialEditor.html)
- `MaterialProperty`: Material properties of the currently selected shader. [Unity - Scripting API: MaterialProperty (unity3d.com)](https://docs.unity3d.com/ScriptReference/MaterialProperty.html)

```csharp
        #region GUI
        // To define a custom shader GUI use the methods of materialEditor to render controls for the properties array.
        public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties) {
            // Check the usability. Well, we will define the `GetFoldoutState` function later
            m_SurfaceOptionsFoldout = GetFoldoutState("SurfaceOptions");

            // https://docs.unity.cn/cn/2021.3/ScriptReference/ShaderGUI.FindProperty.html
            // Returns MaterialProperty if the material property was found, otherwise null.
            m_Test = FindProperty(MPropertyNames.Test, properties, false);
            m_MainTex = FindProperty(MPropertyNames.MainTex, properties, false);
            m_MainColor = FindProperty(MPropertyNames.MainColor, properties, false);

            // Modify and apply the custom GUI
            EditorGUI.BeginChangeCheck(); // https://docs.unity3d.com/ScriptReference/EditorGUI.BeginChangeCheck.html
            DrawProperties(materialEditor);
            // if (EditorGUI.EndChangeCheck()) SetMaterialKeywords(materialEditor.target as Material);
        }

        #endregion
```

As we have archived the usability check in the `OnGUI` function, we just need to archive the `DrawProperties()` function and `GetFoldoutState()` function later.

#### Archive the `GetFoldoutState` function

To archive the folder effects, we need to use the `EditorPrefs` Property to make the target variables a folder first.
[Unity - Scripting API: EditorPrefs (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorPrefs.html)

```C#
        #region EditorPrefs

        // Because we
        private bool GetFoldoutState(string name) {
            return EditorPrefs.GetBool($"{EditorPreKey}.{name}");
        }

        private void SetFoldoutState(string name, bool field, bool value) {
            if (field == value) return;
            EditorPrefs.SetBool($"{EditorPreKey}.{name}", value);
        }

        #endregion
```

#### Setup the foldout render function

Finally, we will archive the function that can truly make effects in the Inspector tab.

Here, we need to define a startup function that controls all the foldouts and then build different functions for every folder. It sounds like a big project. But it is a simple job actually.
[Unity - Scripting API: EditorGUILayout (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUILayout.html)
[Unity - Scripting API: EditorGUILayout.BeginFoldoutHeaderGroup (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginFoldoutHeaderGroup.html)
[Unity - Scripting API: EditorGUILayout.EndFoldoutHeaderGroup (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUILayout.EndFoldoutHeaderGroup.html)

```C#
        #region Properties

        private void DrawProperties(MaterialEditor materialEditor) {
            // the foldout setup here:
            var surfaceOptionsFold =
                EditorGUILayout.BeginFoldoutHeaderGroup(m_SurfaceOptionsFoldout, Styles.SurfaceOptionFold);
            if (surfaceOptionsFold) {
                EditorGUILayout.Space();
                DrawSurfaceOption(materialEditor);
                EditorGUILayout.Space();
            }

            SetFoldoutState("SurfaceOptions", m_SurfaceOptionsFoldout, surfaceOptionsFold);
            EditorGUILayout.EndFoldoutHeaderGroup();
        }

        private void DrawSurfaceOption(MaterialEditor materialEditor) {
            materialEditor.TexturePropertySingleLine(Styles.MainColor, m_MainTex, m_MainColor);

            materialEditor.FloatProperty(m_Test, MPropertyNames.Test);
        }

        #endregion
```

#### [Optional] Setup keywords

Once we change the properties in the custom shader UI, we need to regenerate the corresponding shader variants. Here's an example.

```C#
        #region Keywords

        private void SetKeyword(Material material, string keyword, bool value) {
            if (value) material.EnableKeyword(keyword);
            else material.DisableKeyword(keyword);
        }

        private void SetMaterialKeywords(Material material) {
            // https://docs.unity3d.com/ScriptReference/Material-shaderKeywords.html
            material.shaderKeywords = null;

            SetKeyword(material, "_SPECULAR_SETUP", material.GetFloat(MPropertyNames.Test) == 0);
        }

        #endregion
```

### A Complete Example Script

Find it on: [URP_Toon/ToonShaderGUI.cs at master · ChiliMilk/URP_Toon (github.com)](https://github.com/ChiliMilk/URP_Toon/blob/master/Assets/ChiliMilkToonShader/Editor/ToonShaderGUI.cs)

## Reference

- Blog:

  - [Straw1997/UnityCustomShaderGUI: 自定义 Unity Shader GUI (github.com)](https://github.com/Straw1997/UnityCustomShaderGUI)
  - [simplest way to add an option to right click menu - Unity Forum](https://forum.unity.com/threads/simplest-way-to-add-an-option-to-right-click-menu.424987/)
  - [C# 枚举（Enum） | 菜鸟教程 (runoob.com)](https://www.runoob.com/csharp/csharp-enum.html)
  - [【Unity Shader】自定义材质面板的小技巧*unity shader bool*妈妈说女孩子要自立自强的博客-CSDN 博客](https://blog.csdn.net/candycat1992/article/details/51417965)
  - [Shader 面板上常用的一些内置枚举 UI - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/93194054)

- Unity Manual
  - [Unity - Manual: Using shader keywords with the material Inspector (unity3d.com)](https://docs.unity3d.com/Manual/shader-keywords-material-inspector.html)
  - [Unity - Manual: ShaderLab: assigning a custom editor (unity3d.com)](https://docs.unity3d.com/Manual/SL-CustomEditor.html)
  - [Unity - Scripting API: ShaderGUI.OnGUI (unity3d.com)](https://docs.unity3d.com/ScriptReference/ShaderGUI.OnGUI.html)
  - [Unity - Scripting API: MaterialEditor (unity3d.com)](https://docs.unity3d.com/ScriptReference/MaterialEditor.html)
  - [Unity - Scripting API: MaterialProperty (unity3d.com)](https://docs.unity3d.com/ScriptReference/MaterialProperty.html)
  - [Unity - Scripting API: EditorGUI (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUI.html)
  - [Unity - Scripting API: EditorGUI.BeginChangeCheck (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUI.BeginChangeCheck.html)
  - [Unity - Scripting API: EditorPrefs (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorPrefs.html)
  - [Unity - Scripting API: EditorGUILayout (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUILayout.html)
  - [Unity - Scripting API: EditorGUILayout.BeginFoldoutHeaderGroup (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginFoldoutHeaderGroup.html)
  - [Unity - Scripting API: EditorGUILayout.EndFoldoutHeaderGroup (unity3d.com)](https://docs.unity3d.com/ScriptReference/EditorGUILayout.EndFoldoutHeaderGroup.html)
- [URP_Toon/ToonShaderGUI.cs at master · ChiliMilk/URP_Toon (github.com)](https://github.com/ChiliMilk/URP_Toon/blob/master/Assets/ChiliMilkToonShader/Editor/ToonShaderGUI.cs)
