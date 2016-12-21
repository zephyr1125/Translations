# Object Pooling in Unity

-------------------Part 1 由太昊 翻译-------------------------
![ObjectPooling-feature](http://img.tairan.com/2016-12-19-ObjectPooling-feature-1-250x250.png)

Picture this: You’re testing your latest and greatest shoot ‘em up. Enemies fly on screen as fast as your fingers can tap, then boom! A measly frame skip later and you’re nothing more than a pitiful pile of ash at the hands of some angry, bug-eyed Martian from Neptune!
想象一下：你正在测试你最新的和最棒的一个射击游戏。敌人在以你能掌握的最快速度来回飞行，然后，砰！卡了一帧之后，你就被打成了一个凶神恶煞的外星人手中的一团残渣。
In this scenario, you’re outnumbered 1000 to 1\. There’s no need to give the Neptunians the upper hand with unexpected memory spikes. Sound familiar? Go ahead and pull up a chair and get up to speed on *Object Pooling*.
这可是场以一敌千的战斗，不应该由于预期之外的内存尖峰导致输给外星人。似曾相识不？来，拉把椅子，听听这篇关于*对象池技术*的演说吧。
In this Unity tutorial, you’ll learn:
在这篇Unity教程中，你将学到：
* All about object pooling
* 所有关于对象池技术的内容

* How to pool a game object
* 如何将一个game object入池

* How to expand your object pool at runtime if necessary
* 如何在运行时按需扩展对象池
* How to extend your object pool to accommodate different objects
* 如何扩展对象池以适应不同的对象
By the end of this tutorial, you’ll have a generic script you can readily drop into a new game. Additionally, you’ll understand how to retrofit the same script for an existing game.
在教程的最后，你会得到一个可以直接丢入新游戏的通用脚本。而且，你会懂得如何为现有的游戏改进这个脚本。
*Prerequisites:* You’ll need to be familiar with some basic C# and how to work within Unity’s development environment. If you need some assistance getting up to speed, check out the [Unity tutorials](https://www.raywenderlich.com/unity-tutorials "Unity tutorials") on this site.
*预备知识：*你需要熟悉C#基础并且知道如何操作Unity的开发环境。如果你需要帮助，查看这里[Unity教程](https://www.raywenderlich.com/unity-tutorials "Unity tutorials")
## What is Object Pooling?
## 什么是对象池技术？
`Instantiate()` and `Destroy()` are useful and necessary methods during gameplay. Each generally requires minimal CPU time.
`Instantiate()` 和 `Destroy()` 是在游戏流程中好用而必备的方法。
However, for objects created during gameplay that have a short lifespan and get destroyed in vast numbers per second, the CPU needs to allocate considerably more time.
然而，对于在游戏流程中生命周期短暂而且每秒大量摧毁的对象群而言，CPU进行内存分配的时间十分显著。
![Bullets are a great example of a GameObject that could be pooled.](http://img.tairan.com/2016-12-19-whatthe-650x480.png)
![子弹就是一个适合入池的GameObject的好例子](http://img.tairan.com/2016-12-19-whatthe-650x480.png)
Bullets are a great example of a GameObject that you might pool.
子弹就是一个适合入池的GameObject的好例子
Additionally, Unity uses *Garbage Collection* to deallocate memory that’s no longer in use. Repeated calls to `Destroy()` frequently trigger this task, and it has a knack for slowing down CPUs and introducing pauses to gameplay.
并且，Unity使用*垃圾收集(Garbage Collection)*技术来释放不需要继续使用的内存。不断调用`Destroy()`会频繁的激活收集，而这会拖慢CPU导致游戏流程的卡顿。
This behavior is critical in resource-constrained environments such as mobile devices and web builds.
这一行为在移动设备和网页等资源受限的环境下中尤为致命。
*Object pooling* is where you pre-instantiate all the objects you’ll need at any specific moment before gameplay — for instance, during a loading screen. Instead of creating new objects and destroying old ones during gameplay, your game reuses objects from a “pool”.
*对象池技术*将尚未用到的对象放在游戏流程之前——比如在读取界面的时候——进行预先实例化。这样游戏可以从『池子』中重用对象，而不需在游戏流程中不断地创建和销毁。
![NonVsPooling](http://img.tairan.com/2016-12-19-NonVsPooling.png)
![是否使用池的比对](http://img.tairan.com/2016-12-19-NonVsPooling.png)
## Getting Started
## 开始
If you don’t already have Unity 5 or newer, download it from [Unity’s website](https://unity3d.com/get-unity/download "Unity's website").
如果还没有Unity5或者更新版本，从 [Unity官网](https://unity3d.com/get-unity/download "Unity官网")下载。
Download the [starter project](https://koenig-media.raywenderlich.com/uploads/2016/06/SuperRetroShooter_Starter.zip "starter project"), unzip and open `SuperRetroShooter_Starter` project in Unity — it’s a pre-built vertical scroll shooter.
然后下载 [起始项目](https://koenig-media.raywenderlich.com/uploads/2016/06/SuperRetroShooter_Starter.zip "起始项目")，解压并在Unity里打开 `SuperRetroShooter_Starter`项目——这是一个预先建好的纵向卷轴射击项目。
*Note*: Credit goes to Master484, Marcus, Luis Zuno and Skorpio for the medley of art assets from [OpenGameArt](http://opengameart.org). The royalty-free music was from the excellent [Bensound](http://www.bensound.com "Bensound").
*注意*：艺术资源归功于[OpenGameArt](http://opengameart.org)的 Master484, Marcus, Luis Zuno 和 Skorpio。免授权音乐则来自于杰出的[Bensound](http://www.bensound.com "Bensound")
Feel free to have a look at some of the scripts, such as *Barrier*; they are generic and useful, but their explanations are beyond the scope of this tutorial.
随意看看里面的脚本吧，比如*Barrier*；他们都是通用且实用的，不过这篇教程就不对它们进行解说了。
As you work through everything, it will be helpful to see what’s happening in your game’s Hierarchy during gameplay. Therefore, I recommend that you *deselect Maximize on Play* in the *Game Tab’s toolbar*.
在浏览这些时，注意在游戏流程中Hierarchy里都发生了些什么是很有帮助的。因此我建议取消*Game*页签的工具栏里的*Maximize on Play*选项。
![maximise](http://img.tairan.com/2016-12-19-maximise.png)
![最大化](http://img.tairan.com/2016-12-19-maximise.png)
Click the *play button* to see what you have. :]
点击*play*按钮看看。 :]

Note how there are many `PlayerBullet(Clone)` objects instantiated in the Hierarchy when shooting. Once they hit an enemy or leave the screen, they are destroyed.
注意当发射时，在Hierarchy里实例化了大量的`PlayerBullet(Clone)`。当击中敌人或者离开屏幕时，他们会被销毁。
Making matters worse is the act of collecting those randomly dropped power-ups; they fill the game’s Hierarchy with bullet clones with just a few shots and destroy them all the very next second.
更糟的是收集那些掉落的强化道具，他们迅速用子弹的复制体填满了Hierarchy，随后又在下一秒里立刻销毁它们。
![Youaregoingtofixthat](http://img.tairan.com/2016-12-19-Youaregoingtofixthat.gif)
![你需要修复这里](http://img.tairan.com/2016-12-19-Youaregoingtofixthat.gif)
In its current state, Super Retro Shooter is a “bad memory citizen”, but you’ll be the hero that gets this shooter firing on all cylinders and using resources more scrupulously.
在现在这个状态下，Super Retro Shooter这个游戏就像一个『内存市的坏市民』，而你要做个英雄，让这个游戏全速工作并且更加严谨地使用资源。
### Time to Get Your Feet Wet
### 该实际上手了
Click on the *Game Controller* GameObject in the Hierarchy. Since this object will persist in the Scene, you’ll add your object pooler script here.
在Hierarchy里点击*Game Controller*。这个对象会在场景里持续存在，适合在它上面添加对象池脚本。
In the Inspector, click the *Add Component* button, and select *New C# Script*. Give it the name *ObjectPooler*.
在Inspector中，点击*Add Component*按钮，选择*New C# Script*。起名为*ObjectPooler*。
*Double-click* the new script to open it in MonoDevelop, and add the following code to the class:
*双击*新的脚本在MonoDevelop中打开它，并在类里添加以下代码：
|  

public static ObjectPooler SharedInstance;

void Awake() {

  SharedInstance = this;

}

 |

Several scripts will need to access the object pool during gameplay, and `public static instance` allows other scripts to access it without getting a Component from a GameObject.
有一些脚本会需要在游戏流程中访问对象池，因此用`public static instance`令各个脚本可以访问它而不需要从GameObject上获取Component的操作。
At the top of the script, add the following `using` statement:
在脚本顶端，增加以下`using`声明：
|  

using System.Collections.Generic;

 |

You’ll be `using` a generic list to store your pooled objects. This statement gives you access to generic data structures so that you can use the `List` class in your script.
你将会`使用(using)`一个泛型(generic)列表来存储入池了的对象。这一声明允许你访问泛型数据结构，这样就可以在脚本里用到`List`类。
*Note*: Generic? *Nobody* wants to be generic! Everybody wants to be special!
*注意*:普通(泛型的原文Generic也有普通的意思)？*没人*想要普通！每个人都希望独特！
In a programming language like C#, generics allow you to write code that can be used by many different types while still enforcing type safety.
在程序语言，比如C#里，泛型(普通)能让你的代码可以被很多不同的类型所使用，同时保持类型安全(type safety)。
Typically, you use generics when working with collections. This approach allows you to have an array that only allows one type of object, preventing you from putting a dog inside a cat array, although that could be pretty funny. :]
一个典型的应用就是在使用集合(collection)的时候使用泛型。这个方法限定了在数组(array)中存放对象的类型，以防比如在猫数组里放了个狗，尽管这会很好笑。:]
Speaking of lists, add the object pool list and two new public variables:
说到列表(list)，添加对象池列表以及两个新的公共变量：
|  

public ListGameObject> pooledObjects;

public GameObject objectToPool;

public int amountToPool;

 |

The naming is fairly self-explanatory.
这些变量的命名是很自明的(self-explanatory)。
By using the Inspector in Unity, you’ll be able to specify a GameObject to pool and a number to pre-instantiate. You’ll do that in a minute.
通过Unity的Inspector，你将能够指定GameObject入池，并且设置预实例化的数量。一会儿会做这个。
Meanwhile, add this code to *Start()*:
然后，把这些代码加入*Start()*:
|  

pooledObjects = [new](http://www.google.com/search?q=new+msdn.microsoft.com) ListGameObject>();

for (int i = 0; i  amountToPool; i++) {

  GameObject obj = (GameObject)Instantiate(objectToPool);

  obj.SetActive(false); 

  pooledObjects.Add(obj);

}

 |

The `for` loop will instantiate the `objectToPool` GameObject the specified number of times in `numberToPool`. Then the GameObjects are set to an inactive state before adding them to the `pooledObjects`list.
'for'循环会实例化`numberToPool`所指定的数量的`objectToPool`GameObject。然后这些GameObject会被设置为inactive状态，之后再加入到`pooledObjects`列表中。
Go back to Unity and add the *Player Bullet Prefab* to the *objectToPool* variable in the Inspector. In the *numberToPool* field, type *20*.
回到Unity然后在Inspector中添加*Player Bullet Prefab*到*objectToPool*变量上。在*numberToPool*字段上填入*20*。
![ObjectPoolSetup](http://img.tairan.com/2016-12-19-ObjectPoolSetup.gif)
![对象池设置](http://img.tairan.com/2016-12-19-ObjectPoolSetup.gif)
Run the game again. You should now have 20 pre-instantiated bullets in the Scene with nowhere to go.
再次运行游戏。现在场景里应该有20个预实例化好的没什么地方可去的子弹了。
![Well done! You now have an object pool :]](http://img.tairan.com/2016-12-19-ScreenCap1-208x500.png)

Well done! You now have an object pool. :]
干得好！你现在拥有一个对象池了 :]
### Dipping into the Object Pool
### 深入对象池
Jump back into the *ObjectPooler* script and add the following new method:
回到*ObjectPooler*脚本然后添加以下代码：
|  

public GameObject GetPooledObject() {

//1

  for (int i = 0; i  pooledObjects.Count; i++) {

//2

    if (!pooledObjects[i].activeInHierarchy) {

      return pooledObjects[i];

    }
  
  }

//3 

  return null;

}

 |

The first thing to note is that this method has a `GameObject` return type as opposed to `void`. This means a script can ask for a pooled object from `GetPooledObject` and it’ll be able to return a GameObject in response. Here’s what else is happening here:
首先要注意的是这个方法现在的返回值类型是`GameObject`而非`void`。这是指别的脚本可以向`GetPooledObject`请求一个池中的对象，而他会返回一个GameObject作为回应。其他会发生的事情如下：
1. This method uses a `for` loop to iterate through your `pooledObjects` list.
1. 这个方法使用`for`循环遍历你的`pooledObjects`列表。
2. You check to see if the item in your list is *not* currently active in the Scene. If it is, the loop moves to the next object in the list. If not, you exit the method and hand the inactive object to the method that called `GetPooledObject`.
2. 检查其中一个对象是否`并非`激活(active)状态。如果是激活的，循环到下一个。如果是非激活的，退出方法并把这个对象提交给对`GetPooledObject`的调用者。
3. If there are currently no inactive objects, you exit the method and return nothing.
3. 如果目前没有非激活状态的对象，退出方法并且不返回任何东西。
Now that you can ask the pool for an object, you need to replace your bullet instantiation and destroy code to use the object pool instead.
现在，你可以向池子请求对象了，这需要替换掉原有的子弹的实例化和销毁代码，以对象池替代。
Player bullets are instantiated in two methods in the `ShipController` script.
玩家的子弹会在`ShipController`脚本里的两个方法中实例化。
* `Shoot()`
* `Shoot()`
* `ActivateScatterShotTurret()`
* `ActivateScatterShotTurret()`
Open the *ShipController* script in MonoDevelop and find the lines:
在MonoDevelop里打开*ShipController*脚本：
|  

Instantiate(playerBullet, turret.transform.position, turret.transform.rotation);

 |

Replace *both* instances with the following code:

|  

GameObject bullet = ObjectPooler.SharedInstance.GetPooledObject(); 

  if (bullet != null) {

    bullet.transform.position = turret.transform.position;

    bullet.transform.rotation = turret.transform.rotation;

    bullet.SetActive(true);

  }

 |

*Note*: Make sure this replacement is made in both `Shoot()` and `ActivateScatterShotTurret()`before you continue.
*注意*: 在继续之前，确保在`Shoot()`和`ActivateScatterShotTurret()`方法中也做好了这样的替换。
Previously, the methods iterated through the list of currently active turrets on the player’s ship (depending on power-ups) and instantiated a player bullet at the turrets position and rotation.
在之前，这些方法遍历玩家船上的激活着的子弹列表，然后在枪口位置与角度上实例化子弹。
You’ve set it to ask your `ObjectPooler` script for a pooled object. If one is available, it’s set to the position and rotation of the turret as before, and then set to `active` to rain down fire upon your enemy. :]
而现在改成了询问`ObjectPooler`脚本获取一个池中的对象。如果拿到了，将它设置到枪口的位置与角度，然后设为`active`状态来向敌人倾泻下你的弹火吧：]


------------------PART 1 END----------------------------

------------------PART 2 由 鹿角Kaw-Liga兔 翻译------------

### Get Back in the Pool

Instead of destroying bullets when they’re no longer required, you’ll return them to the pool.

There are two methods that destroy unneeded player bullets:

* `OnTriggerExit2D()` in the `DestroyByBoundary` script removes the bullet when it leaves the screen.
* `OnTriggerEnter2D()` in the `EnemyDroneController` script removes the bullet when it collides and destroys an enemy.

Open *DestroyByBoundary* in MonoDevelop and replace the contents of the `OnTriggerExit2D`method with this code:



|  
if (other.gameObject.tag == "Boundary") {
  if (gameObject.tag == "Player Bullet") {
    gameObject.SetActive(false);
  } else {
    Destroy(gameObject);
  }
}

 |



Here’s a generic script that you can attach to any number of objects that need removal when they leave the screen. You check if the object that triggered the collision has the `Player Bullet` tag — if yes, you set the object to inactive instead of destroying it.

Similarly, open *EnemyDroneController* and find `OnTriggerEnter2D()`. Replace `Destroy(other.gameObject);` with this line of code:



|  
other.gameObject.SetActive(false);

 |



Wait!

Yes, I *can* see you hovering over the play button. After all that coding, you must be itching to check out your new object pooler. Don’t do it yet! There is one more script to modify — don’t worry, it’s a tiny change. :]

Open the *BasicMovement* script in MonoDevelop and rename the `Start` Method to `OnEnable`.

![One gotcha when you use the object pool pattern is remembering the lifecycle of your pooled object is a little different.](http://img.tairan.com/2016-12-19-GameLoop-Diagram.png)

One gotcha when you use the object pool pattern is remembering that the lifecycle of your pooled object is a little different.



Ok, *now* click *play*. :]

As you shoot, the inactive player bullet clones in the Hierarchy become active. Then they elegantly return to an inactive state as they leave the screen or destroy an enemy drone.

*Well Done!*

![ScreenRec4Gif](http://img.tairan.com/2016-12-19-ScreenRec4Gif.gif)

But what happens when you collect all those power-ups?

Running out of ammo eh?

![sgd_16_whathaveyoudone](http://img.tairan.com/2016-12-19-sgd_16_whathaveyoudone-433x320.jpg)

As the game designer, you enjoy supreme powers, such as limiting players’ firepower to encourage a more focused strategy as opposed to just shooting everything and everywhere.

You could exert your powers to do this by adjusting the number of bullets you initially pool to get this effect.

Conversely, you can also go in the other direction pool a huge number of bullets to cover all power-up scenarios. That begs a question: why would you pool 100 bullets for the elusive ultimate power-up when 90 percent of the time 50 bullets is adequate?

You would have 50 bullets in memory that you would only need rarely.

![Fairtome](http://img.tairan.com/2016-12-19-Fairtome-432x320.png)

### The Incredible Expanding Object Pool

Now you’ll modify the object pool so you can opt to increase the number of pooled objects at runtime if needed.

Open the *ObjectPooler* script in MonoDevelop and add this new public variable:



|  
public bool shouldExpand = true;

 |



This code creates a checkbox in the Inspector to indicate whether it’s possible to increase the number of pooled objects.

In `GetPooledObject()`, replace the line `return null;` with the following code.



|  
if (shouldExpand) {
  GameObject obj = (GameObject)Instantiate(objectToPool);
  obj.SetActive(false);
  pooledObjects.Add(obj);
  return obj;
} else {
  return null;
}

 |



If a player bullet is requested from the pool and no inactive ones can be found, this block checks to see it’s possible to expand the pool instead of exiting the method. If so, you instantiate a new bullet, set it to inactive, add it to the pool and return it to the method that requested it.

Click *play* in Unity and try it out. Grab some power-ups and go crazy. Your 20 bullet object pool will expand as needed.

![ScreenRec6Gif](http://img.tairan.com/2016-12-19-ScreenRec6Gif.gif)

![seriously](http://img.tairan.com/2016-12-19-seriously-650x481.png)

-----------------------PART 2 END-------------------------------

-----------------------Part 3 由 翻译 ------------------------
### Object Pool Party

Invariably, lots of bullets mean lots of enemies, lots of explosions, lots of enemy bullets and so on.

To prepare for the onslaught of madness, you’ll extend the object pooler so it handles multiple object types. You’ll take it a step further and will make it possible to configure each type individually from one place in the Inspector.

Add the following code *above* the *ObjectPooler* class:



|  
[System.Serializable]
public class ObjectPoolItem {
}

 |



`[System.Serializable]` allows you to make instances of this class editable from within the Inspector.

Next, you need to move the variables `objectToPool`, `amountToPool` and `shouldExpand` into the new `ObjectPoolItem` class. Heads up: you’ll introduce some errors in the ObjectPooler class during the move, but you’ll fix those in a minute.

Update the `ObjectPoolItem` class so it looks like this:



|  
[System.Serializable]
public class ObjectPoolItem {
  public int amountToPool;
  public GameObject objectToPool;
  public bool shouldExpand;
}

 |



Any instance of `ObjectPoolItem` can now specify its own data set and behavior.

Once you’ve added the above public variables, you need make sure to *delete* those variables from the `ObjectPooler`. Add the following to `ObjectPooler`:



|  
public ListObjectPoolItem> itemsToPool;

 |



This is new list variable that lets you hold the instances of `ObjectPoolItem`.

Next you need to adjust `Start()` of `ObjectPooler` to ensure all instances of `ObjectPoolItem` get onto your pool list. Amend `Start()` so it looks like the code below:



|  
void Start () {
  pooledObjects = [new](http://www.google.com/search?q=new+msdn.microsoft.com) ListGameObject>();
  foreach (ObjectPoolItem item in itemsToPool) {
    for (int i = 0; i  item.amountToPool; i++) {
    GameObject obj = (GameObject)Instantiate(item.objectToPool);
    obj.SetActive(false);
    pooledObjects.Add(obj);
    }
  }
}

 |



In here, you added a new `foreach` loop to iterate through all instances of *ObjectPoolItem* and add the appropriate objects to your object pool.

You might be wondering how to request a particular object from the object pool – sometimes you need a bullet and sometimes you need more Neptunians, you know?

Tweak the code in `GetPooledObject` so it matches the following:



|  
public GameObject GetPooledObject(string tag) {
  for (int i = 0; i  pooledObjects.Count; i++) {
    if (!pooledObjects[i].activeInHierarchy && pooledObjects[i].tag == tag) {
      return pooledObjects[i];
    }
  }
  foreach (ObjectPoolItem item in itemsToPool) {
    if (item.objectToPool.tag == tag) {
      if (item.shouldExpand) {
        GameObject obj = (GameObject)Instantiate(item.objectToPool);
        obj.SetActive(false);
        pooledObjects.Add(obj);
        return obj;
      }
    }
  }
  return null;
}

 |



`GetPooledObject` now takes a `string` parameter so your game can request an object by its `tag`. The method will search the object pool for an inactive object that has a matching tag, and then it returns an eligible object.

Additionally, if it finds no appropriate object, it checks the relevant `ObjectPoolItem` instance by the tag to see if it’s possible to expand it.

### Add the Tags

Get this working with bullets first, then you can add additional objects.

Reopen the *ShipController* script in MonoDevelop. In both `Shoot()` and `ActivateScatterShotTurret()`, look for the line:



|  
GameObject bullet = ObjectPooler.SharedInstance.GetPooledObject();

 |



Append the code so that it includes the `Player Bullet` tag parameter.



|  
GameObject bullet = ObjectPooler.SharedInstance.GetPooledObject(“Player Bullet”);

 |



Return to Unity and click on the *GameController* object to open it in the Inspector.

Add one item to the new *ItemsToPool* list and populate it with 20 player bullets.

![multiobjectpool1](http://img.tairan.com/2016-12-19-multiobjectpool1-405x500.png)

Click *Play* to make sure all that extra work changed nothing at all. :]

Good! *Now* you’re ready to add some new objects to your object pooler.

Change the size of *ItemsToPool* to *three* and add the two types of enemy ships. Configure the *ItemsToPool* instances as follows:

Element 1:

*Object to Pool: EnemyDroneType1
Amount To Pool: 6
Should Expand: Unchecked*

Element 2

*Object to Pool: EnemyDroneType2
Amount to Pool: 6
Should Expand: Unchecked*

![multiobjectpool2](http://img.tairan.com/2016-12-19-multiobjectpool2-341x500.png)

As you did for the bullets, you need to change the `instantiate` and `destroy` methods for both types of ships.

The enemy drones are instantiated in the `GameController` script and destroyed in the `EnemyDroneController` script.

You’ve done this already, so the next few steps will go a little faster. :]

Open the *GameController* script. In *SpawnEnemyWaves()*, find the *enemyType1 instantiation code*:



|  
Instantiate(enemyType1, spawnPosition, spawnRotation);

 |



And replace it with the following code:



|  
GameObject enemy1 = ObjectPooler.SharedInstance.GetPooledObject("Enemy Ship 1"); 
if (enemy1 != null) {
  enemy1.transform.position = spawnPosition;
  enemy1.transform.rotation = spawnRotation;
  enemy1.SetActive(true);
}

 |



Find this *enemyType2 instantiation* code:



|  
Instantiate(enemyType2, spawnPosition, spawnRotation);

 |



Replace it with:



|  
GameObject enemy2 = ObjectPooler.SharedInstance.GetPooledObject("Enemy Ship 2"); 
if (enemy2 != null) {
  enemy2.transform.position = spawnPosition;
  enemy2.transform.rotation = spawnRotation;
  enemy2.SetActive(true);
}

 |



Finally, open the *EnemyDroneController* script. Currently, `OnTriggerExit2D()` just destroys the enemy ship when it leaves the screen. What a waste!

Find the line of code:



|  
Destroy(gameObject);

 |



Replace it with the code below to ensure the enemy goes back to the object pool:



|  
gameObject.SetActive(false);

 |



Similarly, in `OnTriggerEnter2D()`, the enemy is destroyed when it hits a player bullet. Again, find *Destroy()*:



|  
Destroy(gameObject);

 |



And replace it with the following:



|  
gameObject.SetActive(false);

 |



Hit the *play* button and watch as all of your instantiated bullets and enemies change from inactive to active and back again as and when they appear on screen.

![Elon Musk would be very proud of your reusable space ships!](http://img.tairan.com/2016-12-19-ScreenRec8Gif.gif)

Elon Musk would be envious your reusable space ships!



![Finished](http://img.tairan.com/2016-12-19-Finished-650x480.png)

### Where to go From Here?

Thank you for taking the time to work through this tutorial. Here’s a link to the [Completed Project](https://koenig-media.raywenderlich.com/uploads/2016/06/SuperRetroShooter_Complete.zip).

In this tutorial, you retrofitted an existing game with object pooling to save your users’ CPUs from a near-certain overload and the associated consquences of frame skipping and battery burning.

You also got very comfortable jumping between scripts to connect all the pieces together.

If you want to learn more about object pooling, check out [Unity’s live training](https://www.youtube.com/watch?v=9-zDOJllPZ8 "Unity's live training") on the subject.

I hope you found this tutorial useful! I’d love to know how it helped you develop something cool or take an app to the next level.

Feel free to share a link to your work in the comments. Questions, thoughts or improvements are welcome too! I look forward to chatting with you about object pooling, Unity and destroying all the aliens.

----------------------PART 3 END---------------------------

