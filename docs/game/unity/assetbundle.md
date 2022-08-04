# AssetBundle

指 Unity 资源文件打包后的集合

## 作用

根据需要可以将游戏内的资源打包成 `AssetBundle` 文件，可以通过Unity提供的API来对加载和卸载，这样可以减少内存消耗，无需加载所有资源文件。

## 压缩

默认情况下，Unity 通过 `LZMA` 压缩来创建 AssetBundle，然后通过 `LZ4` 压缩将其缓存。[^1]

### LZMA

压缩率高，适合网络传输，但读取其中某个资源需要将整个文件解压。

### LZ4

基于块的压缩算法，压缩率较 `LZMA` 低，但可以在特定块读取资源，是 `AssetBundle` 缓存中用的算法。

打包 AssetBundle 时强制开启 `LZ4` 使用 `BuildAssetBundleOptions.UncompressedAssetBundle`，如：

```c#
[MenuItem("Build Asset Bundles/Uncompressed")]
static void BuildABsUncompressed()
{
    BuildPipeline.BuildAssetBundles("Assets/MyAssetBuilds", BuildAssetBundleOptions.UncompressedAssetBundle, BuildTarget.StandaloneOSX);
}
```

### 缓存

加载的 AssetBundle 以两种方式缓存：内存、硬盘

存储在内存中读取非常快，但会占用大量内存，硬盘则相反。除非需要读取非常频繁，一般都使用硬盘缓存。

可以通过设置 `Caching.compressionEnabled` 来选择是否开启缓存压缩，默认开启，使用 `LZ4` 压缩。

`AssetBundle.LoadFromFile` 或 `AssetBundle.LoadFromFileAsync` 会使用内存缓存，因此应该使用 UWR(UnityWebRequest) API。

如果无法使用 UWR API，可以使用 `AssetBundle.RecompressAssetBundleAsync` 将 LZMA AssetBundle 重新写入硬盘中。

!!! warning "注意"

    WebGL 不支持 LZMA 压缩

[^1]: [官方文档 - AssetBundle 压缩](https://docs.unity3d.com/cn/current/Manual/AssetBundles-Cache.html)