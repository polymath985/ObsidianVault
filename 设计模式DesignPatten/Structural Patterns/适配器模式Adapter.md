---
tags:
  - 设计模式
  - 接口
  - CSharp
---
# 定义

将一个类的接口转换为客户希望的另一个接口。适配器模式将原本由于接口不兼容而不能一起工作的类转变为能够一起工作

```csharp
// 目标接口 - 客户端期望的接口
public interface IMediaPlayer
{
    void Play(string audioType, string fileName);
}

// 被适配者 - 现有的高级媒体播放器
public class AdvancedMediaPlayer
{
    public void PlayVlc(string fileName)
    {
        Console.WriteLine("Playing vlc file. Name: " + fileName);
    }

    public void PlayMp4(string fileName)
    {
        Console.WriteLine("Playing mp4 file. Name: " + fileName);
    }
}

// 适配器类 - 实现目标接口，包含被适配者
public class MediaAdapter : IMediaPlayer
{
    private AdvancedMediaPlayer advancedPlayer;

    public MediaAdapter(string audioType)
    {
        advancedPlayer = new AdvancedMediaPlayer();
    }

    public void Play(string audioType, string fileName)
    {
        if (audioType.Equals("vlc"))
        {
            advancedPlayer.PlayVlc(fileName);
        }
        else if (audioType.Equals("mp4"))
        {
            advancedPlayer.PlayMp4(fileName);
        }
    }
}
```

```csharp
// 客户端使用适配器
public class AudioPlayer : IMediaPlayer
{
    private MediaAdapter mediaAdapter;

    public void Play(string audioType, string fileName)
    {
        // 支持内置格式
        if (audioType.Equals("mp3"))
        {
            Console.WriteLine("Playing mp3 file. Name: " + fileName);
        }
        // 通过适配器支持其他格式
        else if (audioType.Equals("vlc") || audioType.Equals("mp4"))
        {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.Play(audioType, fileName);
        }
        else
        {
            Console.WriteLine(audioType + " format not supported");
        }
    }
}
```

# 角色

- 目标接口：Target，我们期望的接口
- 适配器：Adapter，将被适配者转换为我们期望的形式
- 被适配者：Adaptee，原有的接口

# 优缺点

## 优点

- 可以让任何两个没有关联的类一起运行
- 提高了类的复用
- 通过引入一个适配器类来复用现有的类，无需修改原有结构，遵守了开闭原则

## 缺点

过多的使用适配器，会让系统非常凌乱，不利于整体把握

# 总结

适配器模式属于补偿模式，专门用来在系统后期拓展，修改时使用。因此，适配器模式也不宜过度使用，如果可以的话，我们应该优先通过重构解决。