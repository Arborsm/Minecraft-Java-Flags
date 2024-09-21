<div align="center">
  <a href="https://github.com/kyechan99/capsule-render">
    <img src="https://capsule-render.vercel.app/api?type=waving&height=250&color=timeGradient&text=Minecraft%20Java%20Flags&fontAlignY=46&animation=fadeIn">
  </a>
</div>

<br>

Welcome! this is a guide to (AGGRESSIVELY) tune Java for Minecraft.

> [!NOTE]
> 1. While these tweaks notably reduce some server and client stutters, expect only modest TPS gains + minimal FPS gains at best, and somewhat increased RAM + CPU usage.
> 
> 2. While these flags are easy to copy-paste and forget, they are no substitute for clearing laggy things out with mods like Spark.

<br>

# Picking a Java Distributor
Java runtimes from Azul or Microsoft, Adoptium, Amazon and so on are all basically identical as they are based on OpenJDK. Some notable exceptions are:

1. **GraalVM**  - features a more aggressive Java compiler.

2. **Clear Linux OpenJDK**  - uses a highly compatible OpenJDK base with it's build process and dependencies [optimized for newer CPUs](https://www.phoronix.com/review/zen4-clear-linux/2).

3. **Platform Prime** - ***very*** fast since it hooks into llvm, but is currently incompatible with most mods and Linux-only.

4. **Red Hat Java 8** - has the Shenandoah garbage collector (It is the only Java 8 JDK with it).

5. **OpenJ9** - consumes less memory at the cost of being *much* slower in Minecraft. It also uses totally different flags than any other Java build. This option is unrecommended.

If you dont know what to pick, I recommend GraalVM, Adoptium, or Azul Platform ***Core***. You can download most of the good OpenJDK based ones from [here](https://adoptium.net/marketplace/), and you can download GraalVM from [here](https://www.graalvm.org/downloads/).

<br>

# Picking the correct Java version
While any Java Distributor will work just fine, you may run into compatibility issues depending on the java version you use. Refer to the following table to choose the correct version:

| Minecraft Version | Vanilla + Jarmods | Fabric + Related | Forge + Related |
|-------------------|-------------------|------------------|-----------------|
| 1.19.x and above  | Java 23           | Java 23          | Java 23         |
| 1.16.x - 1.18.x   | Java 23           | Java 23          | Java 23         |
| 1.15.x            | Java 23           | Java 23          | Java 11         |
| 1.14.x            | Java 23           | Java 21          | Java 8          |
| 1.13.x and below  | Java 23           | N/A              | Java 8          |
| 1.12.x - 1.7.10   | Java 23           | N/A              | Java 8          |

<br>

# Memory Allocation
Minimum and maximum memory flags (`-xms` and `-xmx`) should be set to the same value as explained [here](https://dzone.com/articles/benefits-of-setting-initial-and-maximum-memory-siz) with one caveat: if you are on a low-memory system, and Minecraft takes up almost all your RAM, set your minimum memory below your maximum memory to conserve as much as possible. Also try removing these arguments:
```
-XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M
```
 and try server garbage collection arguments.

Sizes are set in megabytes or gigabytes so `-Xms4096M` or `-Xmx8G` are both correct.

> [!NOTE]
> 1. Allocating too much memory can break garbage collection or just slow Minecraft down, even if you have plenty to spare. Allocating too little can also slow down or break the game. Keep a close eye on Task manager (or your DE's system monitor) as Minecraft is running, and allocate only as much as it needs (which is usually less than 8G). `sparkc gcmonitor` will tell you if your allocation is too high (the pauses will be too long) or too low (frequent GC with a low memory warning in the notification).
> 
> 2. If you are using a third-party Minecraft launcher like Prism Launcher or ATLauncher, you shouldn't use memory arguments and instead control memory through the dedicated section in the launcher.

<br>

# Base Java Flags
These are the majority of the flags. The flags below will work for any ***OpenJDK*** 11+ build. They are the same on both servers and clients:

```
-XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysActAsServerClassMachine -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M -XX:-DontCompileHugeMethods -XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000 -XX:+UseVectorCmov -XX:+PerfDisableSharedMem -XX:+UseFastUnorderedTimeStamps -XX:+UseCriticalJavaThreadPriority -XX:ThreadPriorityPolicy=1 -XX:AllocatePrefetchStyle=3
```

For GraalVM Java 17+, use the flags below instead:
```
-XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysActAsServerClassMachine -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:AllocatePrefetchStyle=3 -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M -XX:-DontCompileHugeMethods -XX:+PerfDisableSharedMem -XX:+UseFastUnorderedTimeStamps -XX:+UseCriticalJavaThreadPriority -XX:+EagerJVMCI -Dgraal.TuneInlinerExploration=1
```
For Platform Prime, usually no tuning with special flags is necessary, Large Pages are still applicable though.

And for OpenJ9, use the flags below instead:
```
-XX:+IdleTuningGcOnIdle -XX:+UseAggressiveHeapShrink -XX:-OmitStackTraceInFastThrow -XX:+UseFastAccessorMethods -XX:+OptimizeStringConcat -Xshareclasses:allowClasspaths -Xshareclasses:cacheDir=./cache -Xaot -XX:+UseCompressedOops -XX:ObjectAlignmentInBytes=256 -Xshareclasses -XX:SharedCacheHardLimit=800M -Xtune:virtualized -XX:+TieredCompilation -XX:InitialTenuringThreshold=5 -Dlog4j2.formatMsgNoLookups=true -XX:-DisableExplicitGC -XX:InitiatingHeapOccupancyPercent=35 -XX:+UnlockExperimentalVMOptions -XX:MaxGCPauseMillis=6 -Djava.net.preferIPv4Stack=true -XX:-ParallelRefProcEnabled-XX:+UseTLAB -XX:ReservedCodeCacheSize=70M -XX:G1NewSizePercent=20 -XX:G1ReservePercent=20
```

### Java 8

I *hightly* recommend not using Java 8 unless it is neccesary, but if it is, these flags will work with OpenJDK 8:
```
-XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysActAsServerClassMachine -XX:+ParallelRefProcEnabled -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:+PerfDisableSharedMem -XX:+AggressiveOpts -XX:+UseFastAccessorMethods -XX:MaxInlineLevel=15 -XX:MaxVectorSize=32 -XX:+UseCompressedOops -XX:ThreadPriorityPolicy=1 -XX:+UseDynamicNumberOfGCThreads -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=350M -XX:-DontCompileHugeMethods -XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000 -XX:+UseFPUForSpilling
```

You can also get Java 8 versions of GraalVM EE from the [21.X section on the Oracle site](https://www.oracle.com/downloads/graalvm-downloads.html), for that, use these arguments instead:
```
-XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysActAsServerClassMachine -XX:+ParallelRefProcEnabled -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:+AggressiveOpts -XX:+UseFastAccessorMethods -XX:AllocatePrefetchStyle=1 -XX:ThreadPriorityPolicy=1 -XX:+UseDynamicNumberOfGCThreads -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=350M -XX:-DontCompileHugeMethods -XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000 -XX:+UseFPUForSpilling -XX:+EnableJVMCI -XX:+UseJVMCICompiler -XX:+EagerJVMCI -Dgraal.TuneInlinerExploration=1 -Dgraal.CompilerConfiguration=enterprise -Dgraal.UsePriorityInlining=true -Dgraal.Vectorization=true -Dgraal.OptDuplication=true -Dgraal.DetectInvertedLoopsAsCounted=true -Dgraal.LoopInversion=true -Dgraal.VectorizeHashes=true -Dgraal.EnterprisePartialUnroll=true -Dgraal.VectorizeSIMD=true -Dgraal.StripMineNonCountedLoops=true -Dgraal.SpeculativeGuardMovement=true -Dgraal.InfeasiblePathCorrelation=true
```
Though make sure to set `-Dgraal.VectorizeSIMD` to `false` if you run shaders as this version causes issues with them. This old version also breaks constellation rendering in 1.16.5 Astral Sorcery. This is possibly related to the shader bug. See: https://github.com/HellFirePvP/AstralSorcery/issues/1963

<br>

# Garbage Collection
**Garbage collection flags should be added to Minecraft servers and clients**, as the default "pauses" to stop and collect garbage manifest as stutters on the client and lag on servers. Use the `/sparkc gcmonitor` command in Spark to observe pauses in-game. *Any* old generation pauses are bad, and young generation G1GC collections should be infrequent, but short enough to be imperceptible.  

<br>

### Non-Proactive ZGC 
Non-Proactive ZGC is great for high memory/high core count servers. It has no server throughput hit I can measure, and absolutely does not stutter. However, it requires more RAM and more cores than other garbage collectors. Enable it with
```
-XX:+UseZGC -XX:AllocatePrefetchStyle=1 -XX:-ZProactive
```

> [!NOTE]
> 1. It has a significant client FPS hit.
> 
> 2. Non-Proactive ZGC is unavailable in Java 8 and much less performant in Java 11 than it is in Java 17+.
> 
> 3. Allocate more RAM and more `ConcGCThreads` than you normally would for other GC.
> 
> 4. ZGC doesn't support `AllocatePrefetchStyle=3`, hence setting it to 1 overrides the previous entry. Remove the old one if you want.

<br/>

### Generational ZGC (New and not well tested!)
Generational ZGC is new, so no one has really tested it, though I would assume it's similar to Proactive ZGC, except it also apparently runs well-ish on clients? Enable it with
```
-XX:+UseZGC -XX:AllocatePrefetchStyle=1 -XX:+ZGenerational
```

> [!NOTE]
> 1. Generational ZGC is only available in Java 21+
> 
> 2. Allocate more RAM and more `ConcGCThreads` than you normally would for other GC.
> 
> 3. ZGC doesn't support `AllocatePrefetchStyle=3`, hence setting it to 1 overrides the previous entry. Remove the old one if you want.
>
> 4. GraalVM currently doesn't fully support Generational ZGC as it disables an important optimization (`-XX:+EnableJVMCI`)

<br/>

### Shenandoah
Shenandoah performs well on clients, but kills server throughput. Enable it with 
```
-XX:+UseShenandoahGC -XX:ShenandoahGCMode=iu -XX:ShenandoahGuaranteedGCInterval=1000000 -XX:AllocatePrefetchStyle=1
```

See more tuning options [here](https://wiki.openjdk.org/display/shenandoah/Main). The "herustic" and "mode" options don't change much (except for "compact," which you should not use). 

> [!NOTE]
> 1. Red Hat OpenJDK 8 is the only Java 8 JDK that supports Shenandoah.
> 
> 2. Shenandoah doesn't support `AllocatePrefetchStyle=3`, hence setting it to 1 overrides the previous entry. Remove the old one if you want.
>
> 3. GraalVM currently doesn't support Shenandoah.

<br/>

### Client G1GC
G1GC is the default garbage collector for all JREs. Aikar's [famous Minecraft server G1GC arguments](https://aikar.co/2018/07/02/tuning-the-jvm-g1gc-garbage-collector-flags-for-minecraft/) run great on clients, with two caveats: they effectively [clamp](https://www.oracle.com/technical-resources/articles/java/g1gc.html) the `MaxGCPauseMillis` parameter by setting `G1NewSizePercent` so high, producing long stutters on some clients, and they collect oldgen garbage too aggressively (as the client produces *far* less than a populated server). 

These are similar to the Aikar flags, but with shorter, more frequent pauses, less aggressive G1 mixed collection and more aggressive background collection: 
```
-XX:+UseG1GC -XX:MaxGCPauseMillis=37 -XX:+PerfDisableSharedMem -XX:G1HeapRegionSize=16M -XX:G1NewSizePercent=23 -XX:G1ReservePercent=20 -XX:SurvivorRatio=32 -XX:G1MixedGCCountTarget=3 -XX:G1HeapWastePercent=20 -XX:InitiatingHeapOccupancyPercent=10 -XX:G1RSetUpdatingPauseTimePercent=0 -XX:MaxTenuringThreshold=1 -XX:G1SATBBufferEnqueueingThresholdPercent=30 -XX:G1ConcMarkStepDurationMillis=5.0 -XX:GCTimeRatio=99 -XX:G1ConcRefinementServiceIntervalMillis=150 -XX:G1ConcRSHotCardLimit=16
```

> [!NOTE]
> 1. Java 21+ no longer supports `G1ConcRefinementServiceIntervalMillis` flag and the `-XX:G1ConcRSHotCardLimit=16` flag. Remove them if you want as they will just be ignored.
> 
> 2. `G1NewSizePercent` and `MaxGCPauseMillis` can be used to tune the frequency/dureation of your young generation collections. `G1HeapWastePercent=18` should be removed if you are getting any old generation pauses on your setup. Alternatively, you can raise it and set `G1MixedGCCountTarget` to 2 or 1 to make mixed garbage collection even lazier (at the cost of higher memory usage). 

<br/>

### Server G1GC
Longer pauses are more acceptable on servers. These flags are very close to the aikar defaults:

```
-XX:+UseG1GC -XX:MaxGCPauseMillis=130 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=28 -XX:G1HeapRegionSize=16M -XX:G1ReservePercent=20 -XX:G1MixedGCCountTarget=3 -XX:InitiatingHeapOccupancyPercent=10 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=0 -XX:SurvivorRatio=32 -XX:MaxTenuringThreshold=1 -XX:G1SATBBufferEnqueueingThresholdPercent=30 -XX:G1ConcMarkStepDurationMillis=5.0 -XX:G1ConcRefinementServiceIntervalMillis=150 -XX:G1ConcRSHotCardLimit=16
```

> [!NOTE]
> 1. Java 21+ no longer supports `G1ConcRefinementServiceIntervalMillis` flag and the `-XX:G1ConcRSHotCardLimit=16` flag. Remove them if you want as they will just be ignored.
> 
> 2. `G1NewSizePercent` and `MaxGCPauseMillis` can be used to tune the frequency/dureation of your young generation collections. `G1HeapWastePercent=18` should be removed if you are getting any old generation pauses on your setup. Alternatively, you can raise it and set `G1MixedGCCountTarget` to 2 or 1 to make mixed garbage collection even lazier (at the cost of higher memory usage). 

<br/>

### Garbage Collection Threading
`-XX:ConcGCThreads=[Some Number]` controls the [*maximum* number](https://github.com/openjdk/jdk/blob/dd34a4c28da73c798e021c7473ac57ead56c9903/src/hotspot/share/gc/z/zHeuristics.cpp#L96-L104) of background threads the garbage collector is allowed to use, and defaults to `[number of logical (hyperthreaded) cores / 4]`. Recent versions of Java will [reduce the number of gc threads, if needed](https://wiki.openjdk.org/display/zgc/Main#Main-SettingConcurrentGCThreads).

In some cases (especially with ZGC or Shenandoh) you want to increase this thread cap past the default. I recommend `[number of REAL (non-hyperthreaded) cores - 2]` on most CPUs, but you may need to play with this parameter. If its too low, garbage collection can't keep up with Minecraft, and the game will stutter and/or start eating gobs of RAM and crash. If its too high, it might slow the game down, especially if you are running Java 8. Probably keep it under 8 (you can maybe get away with 10 for ZGC).

No other "threading" flags like `ParallelGCThreads` or `JVMCIThreads` are necessary, as they are enabled by default with good automatic settings in Java 8+.

<br/>

# Large Pages
Enabling large pages improves the performance of Minecraft servers and clients by reducing the load on your system. [Here's a good guide](https://kstefanj.github.io/2021/05/19/large-pages-and-java.html)

> [!NOTE]
> 1. Windows Home doesn't have `gpedit.msc` and thus, can't follow the guide above, intead use [this guide](https://awesomeprojectsxyz.blogspot.com/2017/11/windows-10-home-how-to-enable-lock.html?m=1), also since microsoft took down the tool in mention, you have to download it from [here](https://gist.github.com/eyecatchup/0107bab3d92473cb8a3d3547848fc442).
> 
> 2. On Linux, you generally want to use `-XX:+UseTransparentHugePages`.

Check and see if large pages is working with the `-Xlog:gc+init` java argument in Java 17. 

In any Java version/platform, if large pages isn't working, you will get a warning in the log similar to this: 

`Java HotSpot(TM) 64-Bit Server VM warning: JVM cannot use large page memory because it does not have enough privilege to lock pages in memory.`

<br/>

# SpecialK
A "universal" Windows mod akin to [ReShade](https://reshade.me/), SpecialK has 2 major performance benefits:

- A "smart" frame limiter that reduces stutter, eliminates tearing, saves power, and saves CPU TDP to boost when needed. It even works in conjuction with VRR or Nvidia Reflex. 

- A OpenGL-to-DirectX11 wrapper called OpenGL-IK that eliminates Minecraft's windowed mode overhead, and enables other features (like auto-HDR or a resizable borderless window).

Download it [here](https://www.special-k.info/).

Add your Minecraft launcher. Then navigate to your java bin folder where your javaw.exe is, and create an empty file called `SpecialK.OpenGL32`. Launch your Minecraft launcher with the SpecialK launcher, and the launcher will then "inject" SpecialK into Minecraft.
![SpecialK](specialk.png)

You can create a desktop shortcut to your Minecraft launcher through the SpecialK UI for even more convenience. 

Be sure to turn off VSync and the in-game Minecraft frame limiter.

<br>

# Process Priority
After launching Minecraft, set Java to run at an "Realtime" process priority in Windows with the Task Manager in the details tab:

![Task Manager](taskmgr.png)

Linux users can add  `sudo nice -n -19` to the beginning of the launch command.

> [!CAUTION]
> On Linux, nice levels below 0 (with the "max" being -20) require running nice as `root` which in turn runs Minecraft as `root`. This is a security risk.
> Workarounds:
> 1. Start game and run `renice` as `root` instead with the pid of the minecraft instance. This is safe as Minecraft itself is not running as `root`.
> 2. Modify `/etc/security/limits.conf` to allow non-root users to go to -19. Then just use the normal nice command without `sudo`.
> 3. "Hacky" way to script it: `sudo nice -n -19 su <username> -c`. The easiest way but the first 2 are probably better.

<br/>

# Performance Mods
If you want pure performance, don't use Optifine. There are better performing mods like [Sodium](https://modrinth.com/mod/sodium). Sodium also supports Fabric natively (unlike Optifine) and the 0.6 beta supports NeoForge aswell.

A Good Example with all the performance mods you'll need it [Simply Optimized](https://modrinth.com/modpack/sop).
> [!NOTE]
> It seems that Simply Optimized's list of mods in the description is out of date, instead check what mods come with the version of the modpack you are downloading.

If you want a up-to-date Optifine replacement, you can instead take a look at [Fabulously Optimized](https://modrinth.com/modpack/fabulously-optimized).

<br/>

# Other Performance Tips
- Run your Minecraft on Linux! In pretty much every scenario it runs very good, for servers, Clear Linux and RHEL are very well optimized, and for clients, CachyOS and Arch are amazing.

- Make sure the Minecraft client is using your discrete GPU! Check the F3 tab, and force Minecraft to use it in the "**Windows Graphics Settings**", *not* the AMD/Nvidia control panel (as they don't seem to work anymore).

- Minecraft client Linux users should research running Minecraft natively on wayland if you use it.

- Close everything in the background, including Discord, game launchers and your browser! Minecraft is resource intensive, and does not like other apps generating CPU interrupts or eating disk I/O, RAM and so on.

- Server owners can check this out: [YouHaveTrouble/minecraft-optimization](https://github.com/YouHaveTrouble/minecraft-optimization).

<br/>

# Frequently Asked Questions
- Java tweaks improve server performance and client stuttering, but they don't boost average client FPS much (if at all). For that, running [correct/up-to-date graphics drivers](https://github.com/CaffeineMC/sodium-fabric/wiki/Driver-Compatibility) and [performance mods](https://github.com/TheUsefulLists/UsefulMods) is far more important.

- IBM's OpenJ9 does indeed save RAM, as its reputation would suggest, but is over 30% slower at server chunkgen in my tests. If there are any flags that make it competitive with OpenJDK, please let me know on Discord or make a issue.

<br/>

# Flag Explanations
- Aikar G1GC flags are explained [here](https://aikar.co/2018/07/02/tuning-the-jvm-g1gc-garbage-collector-flags-for-minecraft/)

- `-XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions` simply unlock more flags to be used. These can be listed with the `-XX:+PrintFlagsFinal` and `-XX:+JVMCIPrintProperties` flags.

- `-XX:G1MixedGCCountTarget=3`: This is how many oldgen GC blocks to target in "mixed" GC. These mixed collections are much slower, and the Minecraft client doesn't generate oldgen very quickly, so we can lower this value to 3, 2, or even 1 for shorter GC pauses.

- `-XX:-DontCompileHugeMethods` *Allows* huge methods to be compiled. Modded Minecraft has some of these, and we don't care about higher background compiler CPU usage.

- `-XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000` Enable optimization of larger methods. See [Java Bug #8058148](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8058148).

- `-XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M` reserves more space for compiled code. All sections must "add up" to `ReservedCodeCacheSize`. I have observed modded Minecraft run into the default 250 megabyte limit with `XX:+PrintCodeCache`, but even if its not filled, the larger size makes eviction of compiled code less aggressive.

- `-XX:NmethodSweepActivity=1` (default 10) keeps "cold" code in the cache for a longer time. There is no risk of "filling up" the code cache either, as cold code is more aggressively removed as it fills up.

- ~~`-XX:+UseStringDeduplication`~~ This is a popular option, but not used here, as it's benching slower. Maybe its useful on low memory systems?

- `-XX:+UseFastUnorderedTimeStamps` Avoid system calls for getting the time. The impact of this will vary per system, but we aren't really concerned with logging timestamp accuracy.

- `-XX:+UseCriticalJavaThreadPriority` *Nothing* should preempt the Minecraft threads. GC and Compiler threads can wait.

- `-XX:ThreadPriorityPolicy=1` Use a wider range of thread priorities. Requires sudo on linux to work. Some JDKs (like Graal) enable this by default, but some don't.

- `-XX:G1SATBBufferEnqueueingThresholdPercent=30 -XX:G1ConcMarkStepDurationMillis=5 -XX:G1ConcRefinementServiceIntervalMillis=150`: Optimizes G1GC's concurrent collection threads, still being tested [here](https://research.spec.org/icpe_proceedings/2014/p111.pdf).

- `-XX:G1RSetUpdatingPauseTimePercent=0`: We want *all* this work to be done in the G1GC concurrent threads, not the pauses.

- `-XX:G1HeapWastePercent=18` Don't bother collecting from old gen until its above this percent. This avoids triggering slower "mixed" young generation GCs, which is fine since Minecraft (with sufficient memory) doesn't fill the old gen that fast. Idea from [r/Minecraft/comments/k9zb7m](https://www.reddit.com/r/Minecraft/comments/k9zb7m/tuning_jvm_gc_for_singleplayer/).

- `-XX:GCTimeRatio=99` As a goal, 1% of CPU time should be spent on garbage collection. Default is 12, which seems way too low. The default for Java 8 was 99.

- `-XX:AllocatePrefetchStyle=3` Generate one prefetch instruction per cache line. More aggressive prefetching is generally useful on newer CPUs with large caches. It seems to break ZGC. See [OpenJDK's macro.cpp](https://github.com/openjdk/jdk/blob/bd90c4cfa63ba2de26f7482ed5d1704f9be9629f/src/hotspot/share/opto/macro.cpp#L1806).

- `-Dgraal.LoopRotation=true` A non default optimization, will probably be default soon.

- `-Dgraal.TuneInlinerExploration=1` Spend more time making inlining decisions. For Minecraft, we want the C2 compiler to be as slow and aggressive as possible.

- Most other `-Dgraal` arguments are enabled by default, and are either there as a sanity check, for debugging or as a failsafe (if, for instance, someone unknowingly disables JVCMI with some other flag).

- Many Java 8 flags (such as `-XX:MaxInlineLevel=15 -XX:MaxVectorSize=32`) are just copied from the Java 17 defaults. Others (like `+AggressiveOpts`) are only non-default in some older Java 8 builds.

<br>

# Unmentioned Sources
- Updated Aikar flags from this repo: [etil2jz/etil-minecraft-flags](https://github.com/etil2jz/etil-minecraft-flags)

- Reddit post from a Forge dev: [r/feedthebeast/comments/5jhuk9](https://www.reddit.com/r/feedthebeast/comments/5jhuk9/modded_mc_and_memory_usage_a_history_with_a)

- GraalVM release notes: [graalvm.org/release-notes](https://www.graalvm.org/release-notes)

- Oracle's Java 17 Documentation: [docs.oracle.com/en/java/javase/17/docs/specs/man/java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html)

- VM Options explorer: [chriswhocodes.com](https://chriswhocodes.com)

- Java itself, via the `-XX:+PrintFlagsFinal` and the `-XX:+JVMCIPrintProperties` flags to dump flag descriptions/defaults.

- OpenJDK source: [openjdk/jdk](https://github.com/openjdk/jdk/)

- Testing from @keyboard.tn in Discord.

- [research.spec.org/icpe_proceedings/2014/p111.pdf](https://research.spec.org/icpe_proceedings/2014/p111.pdf)

- [diva-portal.org/smash/get/diva2:1466940/FULLTEXT01.pdf](https://www.diva-portal.org/smash/get/diva2:1466940/FULLTEXT01.pdf)

- [malloc.se/blog/zgc-jdk17](https://malloc.se/blog/zgc-jdk17)

- [docs.oracle.com/javase/8/embedded/develop-apps-platforms/codecache](https://docs.oracle.com/javase/8/embedded/develop-apps-platforms/codecache.htm)
