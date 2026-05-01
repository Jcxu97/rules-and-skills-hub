---
name: game-modding
description: Use when developing game mods, trainers, cheats, or patches — covers Unity/IL2CPP Harmony patching, memory hacking, DLL injection, config modding, and reverse engineering workflows
---

# Game Modding & Trainer Development

## 适用场景

- Unity 游戏的 Harmony/MelonLoader/BepInEx mod 开发
- IL2CPP 游戏的逆向与 patch
- 内存修改器 / Trainer（Cheat Engine 脚本、C# trainer）
- 游戏配置/存档修改
- DLL 注入与 hook
- Unreal Engine blueprint/pak mod

## 工作流

### 1. 侦察（Recon）

开始修改前必须先搞清楚目标：

```
目标确认清单：
- [ ] 游戏引擎（Unity Mono / Unity IL2CPP / Unreal / 自研）
- [ ] Mod 框架（MelonLoader / BepInEx / Harmony standalone / 无）
- [ ] 目标行为（要改什么：数值、逻辑、UI、物理）
- [ ] 反作弊（EAC / BattlEye / VAC / 无 — 单机不管）
- [ ] 已有 mod 生态（有没有人做过类似的）
```

**Unity Mono 游戏：** 直接用 dnSpy/ILSpy 反编译 `Assembly-CSharp.dll`，找目标类和方法。

**Unity IL2CPP 游戏：** 用 Il2CppDumper 导出 `dump.cs`，结合 Ghidra/IDA 定位 RVA，用 Harmony 或 unhollowed assembly patch。

**Unreal 游戏：** pak 解包找 Blueprint/DataTable，或用 UE4SS Lua 脚本。

### 2. 定位目标方法

找到要 patch 的具体方法：

```csharp
// 典型目标示例
PlayerManager.GetCaloriesRemainingToday()    // 修改饥饿
WeatherTransition.ChooseNextWeather()        // 控制天气
ConditionTable.GetDecayPerHour()             // 物品不腐烂
GunItem.GetRecoilAmount()                    // 无后坐力
```

**搜索策略：**
1. 从游戏行为反推 → 搜关键词（Hunger, Health, Damage, Recoil）
2. 看反编译的类继承关系 → 找 Manager/Controller 类
3. 查已有 mod 源码 → 看别人 patch 了哪些方法
4. 运行时 log → 用 Harmony Transpiler 插日志确认调用链

### 3. 编写 Patch

#### Harmony Prefix/Postfix 模板

```csharp
[HarmonyPatch(typeof(TargetClass), nameof(TargetClass.TargetMethod))]
internal class Patch_TargetMethod
{
    // Prefix: 方法执行前拦截，return false 跳过原方法
    static bool Prefix(TargetClass __instance, ref float __result)
    {
        if (!ModSettings.Enabled) return true; // toggle 开关
        __result = 999f;
        return false; // 跳过原方法
    }

    // Postfix: 方法执行后修改返回值
    static void Postfix(ref float __result)
    {
        if (ModSettings.Enabled)
            __result *= 0f; // 例：无后坐力
    }
}
```

#### Harmony Transpiler 模板（修改 IL）

```csharp
[HarmonyPatch(typeof(TargetClass), nameof(TargetClass.TargetMethod))]
internal class Transpiler_TargetMethod
{
    static IEnumerable<CodeInstruction> Transpiler(IEnumerable<CodeInstruction> instructions)
    {
        var codes = new List<CodeInstruction>(instructions);
        for (int i = 0; i < codes.Count; i++)
        {
            // 找到目标指令并替换
            if (codes[i].opcode == OpCodes.Callvirt &&
                codes[i].operand is MethodInfo m &&
                m.Name == "GetDamage")
            {
                // 替换为返回 9999
                codes[i] = new CodeInstruction(OpCodes.Ldc_R4, 9999f);
            }
        }
        return codes;
    }
}
```

### 4. Toggle 系统

**所有修改必须可开关。** 不可开关的 mod 是垃圾。

```csharp
public static class ModSettings
{
    public static bool InfiniteHealth = false;
    public static bool NoRecoil = false;
    public static bool InfiniteAmmo = false;
    // ... 每个功能独立开关
}
```

UI 绑定用 `KeyCode` 热键或游戏内菜单（UnityExplorer / 自建 IMGUI / uGUI overlay）。

### 5. 常见修改模式速查

| 目标 | 方法 |
|------|------|
| 无敌 | Prefix `TakeDamage` → return false |
| 无限弹药 | Postfix `GetAmmoCount` → __result = maxAmmo |
| 无后坐力 | Postfix `GetRecoil` → __result = 0 |
| 加速 | Postfix `GetMoveSpeed` → __result *= multiplier |
| 瞬移 | 直接写 Transform.position |
| 无冷却 | Prefix `StartCooldown` → return false |
| 物品不消耗 | Prefix `ConsumeItem` → return false |
| 修改掉落 | Transpiler 改 loot table 概率 |
| 透视/ESP | OnGUI/IMGUI 画 WorldToScreen 框 |
| 自瞄 | Update 里算最近敌人角度 → 写 camera rotation |

### 6. 调试与验证

```
验证清单：
- [ ] 编译通过（0 error, 0 warning）
- [ ] 游戏正常启动，不崩溃
- [ ] Toggle ON → 效果生效
- [ ] Toggle OFF → 恢复原版行为
- [ ] 不影响其他 mod（检查 Harmony patch 冲突）
- [ ] 存档加载/保存不报错
```

**调试手段：**
- `MelonLogger.Msg()` / `BepInEx.Logging`
- Unity `Player.log` / `output_log.txt`
- Harmony `HarmonyFileLog.Enabled = true`
- dnSpy attach 调试（Mono build only）

### 7. 性能注意

- **Prefix/Postfix** 每帧调用的方法（Update/FixedUpdate）要极轻量
- **避免在 patch 里 GC 分配**（不 new、不 LINQ、不 string concat）
- **高频 patch 考虑动态挂载**：只在 toggle ON 时 `Harmony.Patch()`，OFF 时 `Harmony.Unpatch()`
- **IL2CPP 的 Harmony bridge 有固定开销**：patch 总数越少越好

### 8. 与已有 Mod 共存

开发前先查目标方法有没有被其他 mod patch：

```csharp
var patches = Harmony.GetPatchInfo(targetMethod);
if (patches != null)
{
    // patches.Prefixes / patches.Postfixes / patches.Transpilers
    // 检查 owner, priority, before/after
}
```

**冲突解决：**
- 设置 `[HarmonyPriority(Priority.High)]` 或 `[HarmonyAfter("other.mod.id")]`
- 如果都是 Postfix 修改同一个 `__result`，后执行的 wins
- Transpiler 冲突最难解，尽量避免对同一方法多个 Transpiler

## 反模式

1. **硬编码偏移量** — 游戏更新就废；用方法名/签名匹配
2. **无 toggle 的全局修改** — 没法关闭 = 没法调试
3. **patch Update() 做重逻辑** — 性能炸裂；用事件驱动
4. **忽略 null check** — 游戏状态比你想的复杂，加载/切场景时对象可能不存在
5. **不看已有 mod** — 重复造轮子 + patch 冲突

## TLD (The Long Dark) 特定

基于你的 TldHacks 经验：

- Mod 框架：MelonLoader 0.6.x
- 核心 DLL：`Assembly-CSharp.dll`（Mono build）
- 常用命名空间：`Il2Cpp`（MelonLoader unhollowed）
- catalog.json：`GEAR_` 前缀的物品 prefab 名
- 存档位置：`%APPDATA%\..\LocalLow\Hinterland\TheLongDark\`
- 已知坑：高频 Harmony patch 总数影响帧数（DynamicPatch 模式解决）
- 冲突检查：UniversalTweaks / SonicMode / Sprainkle 等
