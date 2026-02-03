# GDSharp (GD#) Readme

_snake_case and SCREAMING_CONSTANTS are brutalist guidelines for code.
GDSharp applies C# styleguides for GDScript built-in types via wrappers.

GDSharp is something I started on a whim because I couldn't scan snake_case,
it all looked like noise to me. Colored syntax highlighting actually makes it worse.
It's definitely not for everyone, but for modern-day C# developers.

## Node .. uhm Note!

I don't plan on building this to completion. I just add(ed) what I need(ed).
And I only write GDScript where I absolutely HAVE to. Or maybe if I have some spare time ..

Still - I'd love to see GDSharp become an alternative fact! 
It shows that GDScript does not need to look like a snake language.

Thus: Contributions and pull requests are VERY welcome!!

But please: Don't waste time writing all those wrappers manually! 
Spend some AI credits and get it done >=10x faster, at least the first pass.

## Code Example 

This is what my [LunyScript](https://lunyscript.com) plugin.gd looks like:

    @tool
    extends EditorPlugin

    # usings
    const File = Gds.File
    const Str = Gds.Str
    const Res = Gds.Res
    const Project = GdsEditor.Project

    const LunyEngineAutoloadName := "LunyEngineGodotAdapter"
    const LunyGodotAdapterUid := "uid://ss4vx144dk5g" # UID of LunyEngineGodotAdapter.cs
    const AnalyzerPath = "addons/lunyscript/LunyScript/Analyzers/LunyScript-Analyzers.dll"

    func AddLunyEngineAutoload() -> void:
        var resPath := Res.UidToPath(LunyGodotAdapterUid)
        Project.AddAutoloadSingleton(self, LunyEngineAutoloadName, resPath)
        Project.Save()

    func RemoveLunyEngineAutoload() -> void:
        Project.RemoveAutoloadSingleton(self, LunyEngineAutoloadName)
        Project.Save()

    func EnsureAnalyzerReferencedInCsproj() -> void:
        var path = GetCsprojPath()
        if path == "":
            return
            
        var file = File.OpenReadable(path)
        if not file:
            return
            
        var content = File.ReadAllText(file)
        File.Close(file)
        
        # Normalize slashes for comparison to avoid duplicates if different slashes are used
        var checkPath = Str.Replace(AnalyzerPath, "\\", "/")
        var normalizedContent = Str.Replace(content, "\\", "/")
        
        if Str.Contains(normalizedContent, "<Analyzer Include=\"" + checkPath + "\""):
            return
            
        # Insert before </Project>
        var insertPos = Str.Find(content, "</Project>")
        if insertPos == -1:
            return
            
        var block = "\n  <ItemGroup>\n    <Analyzer Include=\"" + AnalyzerPath + "\" />\n  </ItemGroup>\n"
        var newContent = Str.Insert(content, insertPos, block)
        
        file = File.OpenWritable(path)
        if file:
            File.WriteAllText(file, newContent)
            File.Close(file)

    func GetCsprojPath() -> String:
        var assemblyName = Project.GetSetting("dotnet/project/assembly_name", "")
        if assemblyName == "":
            assemblyName = Project.GetSetting("application/config/name", "")

        var path = "res://" + assemblyName + ".csproj"
        if File.Exists(path):
            return path

        # TODO: Fallback for multiple .csproj

        return ""


    ## Wrapped Signals ...
    func OnEnablePlugin() -> void:
        AddLunyEngineAutoload()
        EnsureAnalyzerReferencedInCsproj()

    func OnDisablePlugin() -> void:
        RemoveLunyEngineAutoload()

    func OnEnterTree() -> void:
        AddLunyEngineAutoload()
        EnsureAnalyzerReferencedInCsproj()

    func OnBuild() -> void:
        EnsureAnalyzerReferencedInCsproj()

    func OnEditorReceiveFocus() -> void:
        EnsureAnalyzerReferencedInCsproj()


    ## GDScript Signals ...
    func _enable_plugin() -> void:
        OnEnablePlugin()

    func _disable_plugin() -> void:
        OnDisablePlugin()

    func _enter_tree() -> void:
        OnEnterTree()

    func _notification(what: int) -> void:
        if what == NOTIFICATION_APPLICATION_FOCUS_IN:
            OnEditorReceiveFocus()

    func _build() -> bool:
        OnBuild()
        return true
