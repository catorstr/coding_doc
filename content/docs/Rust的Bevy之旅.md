# Bevy的旅程

> 基于bevy = "0.11.2"的例子
>
> cargo new demo

```toml
#Cargo.toml

[package]
name = "demo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

# [dependencies]
# bevy ="0.11.2"


#这些只是为了在开发时体验更好
[profile.dev]
opt-level = 1
[profile.dev.package."*"]
opt-level = 3
[dependencies]
bevy ={version =  "0.11.2",features = ["dynamic_linking"]} #指定bevy版本，并使用动态链接的方式


```

### bevy demo

```rust
//main.rs
use bevy::{prelude::*, core_pipeline::clear_color::ClearColorConfig};
fn main() {
    App::new()
    .add_plugins(
        DefaultPlugins//添加默认插件
        .set(ImagePlugin::default_nearest())//默认的图片的分辨率很差，这里设置图片分辨率为最接近原图
        .set(WindowPlugin{
            primary_window:Some(Window{
                title:"name".into(), //设置窗口名称
                resolution:(640.0,680.0).into(),//窗口大小
                resizable:false,
                ..default()
            }),
            ..default()
        })
        .build(),
    )
    .add_systems(Startup, setup)
    .add_systems(Update, character_movement)
    .run();
}

fn setup(mut command:Commands,asset_server:Res<AssetServer>){
    //获取一个2D相机
    command.spawn(Camera2dBundle::default());
    //更改相机的默认颜色
    // command.spawn(Camera2dBundle{
    //     camera_2d:Camera2d { clear_color: ClearColorConfig::Custom(Color::GREEN),
  
    //     },
    //     ..default()
    // });
    //加载静态资源--加载一个图像
    //默认bevy会在文件夹assets下查找
    let texture = asset_server.load("sprites/star.png");
    //添加精灵
    command.spawn(SpriteBundle{
        sprite:Sprite{
            custom_size:Some(Vec2::new(100.0, 100.0)), //默认是一个白色的
            ..default()
        },
        texture,//将图片添加到这里
        ..default()
    });
}

fn character_movement(
    mut characters:Query<(&mut Transform,&Sprite)>,
    input:Res<Input<KeyCode>>,//获取键盘信息
    time:Res<Time>,
){
    for(mut transform,_)in &mut characters{
        if input.pressed(KeyCode::W){
            transform.translation.y +=100.0*time.delta_seconds();
        }
        if input.pressed(KeyCode::S){
            transform.translation.y -=100.0*time.delta_seconds();
        }
        if input.pressed(KeyCode::D){
            transform.translation.x +=100.0*time.delta_seconds();
        }
        if input.pressed(KeyCode::A){
            transform.translation.x -=100.0*time.delta_seconds();
        }
    }
}

```

```rust
use bevy::{prelude::*, render::camera::ScalingMode, core_pipeline::clear_color::ClearColorConfig};
fn main() {
    App::new()
        .add_plugins(
            DefaultPlugins //添加默认插件
                .set(ImagePlugin::default_nearest()) //默认的图片的分辨率很差，这里设置图片分辨率为最接近原图
                .set(WindowPlugin {
                    primary_window: Some(Window {
                        title: "name".into(),              //设置窗口名称
                        resolution: (640.0, 680.0).into(), //窗口大小
                        resizable: false,
                        ..default()
                    }),
                    ..default()
                })
                .build(),
        )
        .insert_resource(Money(100.0))//将资源添加到“世界”中
        .add_systems(Startup, setup)
        // .add_systems(Update,spawn_egg)
        // .add_systems(Update, egg_lifetime)
        // .add_systems(Update, character_movement)
        //当然也可以简化代码
        .add_systems(Update, (spawn_egg,egg_lifetime))
        .add_systems(Update, (spawn_egg,egg_lifetime,character_movement))
        .run();
}

fn setup(mut command: Commands, asset_server: Res<AssetServer>) {
    //获取一个2D相机
    //command.spawn(Camera2dBundle::default());

   // 更改相机的默认颜色
    command.spawn(Camera2dBundle{
        camera_2d:Camera2d { clear_color: ClearColorConfig::Custom(Color::GREEN),
        },
        ..default()
    });

    // //设置相机模式
    // let mut camera = Camera2dBundle::default();
    // camera.projection.scaling_mode = ScalingMode::AutoMin {
    //     min_width: 256.0,
    //     min_height: 144.0,
    // };//带来的变化就是，我们的“精灵”看起来变大了
    // command.spawn(camera);

    //加载静态资源--加载一个图像
    //默认bevy会在文件夹assets下查找
    let texture = asset_server.load("sprites/star.png");

    // command.spawn((
    //     SpriteBundle {
    //         sprite: Sprite {
    //             //添加精灵
    //             custom_size: Some(Vec2::new(100.0, 100.0)), //精灵所在的位置，默认是一个白色的
    //             ..default()
    //         },
    //         texture, //将图片添加到这里
    //         ..default()
    //     },
    //     Player { speed: 100.0 }, //添加玩家
    // ));
    //等价写法：
    command
        .spawn(SpriteBundle {
            sprite: Sprite {
                custom_size: Some(Vec2::new(100.0, 100.0)),
                ..default()
            },
            texture,
            ..default()
        })
        .insert(Player { speed: 100.0 });
}

fn character_movement(
    //mut characters:Query<(&mut Transform,&Sprite)>,
    mut characters: Query<(&mut Transform, &Player)>, //将查询变为只匹配我们的“玩家”，在世界中
    input: Res<Input<KeyCode>>,                       //获取键盘信息
    time: Res<Time>,
) {
    // for(mut transform,_)in &mut characters{
    for (mut transform, player) in &mut characters {
        //将“精灵”速速修改为玩家的速度
        let movement_amount = player.speed * time.delta_seconds();
        if input.pressed(KeyCode::W) {
            transform.translation.y += movement_amount // 100.0 * time.delta_seconds();
        }
        if input.pressed(KeyCode::S) {
            transform.translation.y -= movement_amount //100.0 * time.delta_seconds();
        }
        if input.pressed(KeyCode::D) {
            transform.translation.x += movement_amount // 100.0 * time.delta_seconds();
        }
        if input.pressed(KeyCode::A) {
            transform.translation.x -= movement_amount // 100.0 * time.delta_seconds();
        }
    }
}

//创建玩家组件
#[derive(Component)]
pub struct Player {
    pub speed: f32,
}

//创建游戏资源
#[derive(Resource)]
pub struct Money(pub f32);//玩家的钱 f32类型，多出的会“溢出”

//bevy有一个trait，允许我们创建可以访问整个ECS“世界”的资源
//这对于渲染之类的事情非常有用，你可能需要访问渲染设备，但它更多的是一个中间功能
// pub trait FromWorld{
//     fn from_world(world:&mut World)->Self;
// }


//现在我们添加游戏行为：
// 当我们按下“空格”键时，消耗“玩家”10块钱，生成一个蛋，
//过一段时间后，蛋成熟并消失，玩家收货15块钱。

#[derive(Component)]
pub struct Egg{
    pub lifetime:Timer,
    //bevy 为我们创建了一个用于计时器的工具
}

fn spawn_egg(
    mut command:Commands,
    asset_server:Res<AssetServer>,//需要访问静态资源
    input:Res<Input<KeyCode>>,//需要获取键盘信息
    mut money:ResMut<Money>,//需要更改我们的游戏资源
    player:Query<&Transform,With<Player>>,//需要从世界中查询(获取)玩家,
    //此处With是一个过滤器
   //Query 实际上需要两个参数：一个用于我们想要的数据，另一个用于过滤查询
   //查询有多种过滤方法，最常见的就是有约束和无约束 及 With和 WithOut
   //我们的过滤器和数据请求都可以是元组
){
    //如果没有按下 空格键此处返回
    if !input.just_pressed(KeyCode::Space){
        return;
    }

    //接下来 获取玩家变换
    //single 当只有一个实体与查询匹配时，返回单个只读查询项
    let player_transform = player.single();
    //player.get_single()
    //当只有一个实体与查询匹配时，返回单个只读查询项。
    //如果查询项的数目不正好是一个，则返回一个QuerySingleError。

    //处理游戏行为
    if money.0>=10.0{
        money.0-=10.0;
        info!("spent ￥10 on a egg,remaining money:￥{:?}",money.0);
    }else{
        return;//钱不够了
    }
  
    //最后我们加载“蛋精灵”，并在玩家的变换中，产出一个“蛋”。
    let texture = asset_server.load("sprites/ball_blue_large_alt.png");
    command.spawn(
        (SpriteBundle{
            texture,
            transform:*player_transform,
            ..default()
        },
        Egg{
            //当我们“下蛋时”，间计时器设置为5s，并只执行一次
            lifetime:Timer::from_seconds(5.0, TimerMode::Once)
        }
    ));
}

//计时器不会自动计时，我们有责任让它执行计时任务
//创建一个让蛋消失的寿命的系统
fn egg_lifetime(
    mut command:Commands,
    time:Res<Time>,
    mut eggs:Query<(Entity,&mut Egg)>,//这里的Entity是Egg的id号，这将用于command去操作对应的egg消失
    mut money:ResMut<Money>,
){

    //在这个系统中，我们循环所有的Egg组件，并勾选它的“生命周期计时器”

    for(egg_entity,mut egg)in &mut eggs{
       //time.delta()以持续时间的形式返回自上次更新以来前进了多少时间。
       //将计时器提前delta秒。非重复计时器将在持续时间钳制。重复计时器将环调试绕。不会影响暂停的计时器。
        //说人话，若lifetime=5 则倒计时5s后，又重复循环，当计时器是一次性计时器时不循环倒计时。
        egg.lifetime.tick(time.delta());
        //因为我们使用的是一次性计时器，使用这里不会循环，倒计时完之后继续往下一行执行
        if egg.lifetime.finished(){//当计时任务结束时
            //让Egg实体消失(即页面上产生的“蛋精灵”消失)
            command.entity(egg_entity).despawn();//消除实体
            money.0+=15.0;
            info!("Egg sold for ￥15 Current money:￥{:?}",money.0);
        }
    }
}
```

### bevy base

```rust
use bevy::prelude::*;
fn main() {
    App::new()
    .add_systems(Startup, setup)//Startup 只在系统启动时执行一次
  
    //.add_plugins(DefaultPlugins) //DefaultPlugins插件 默认是一个黑窗口,并且提供了游戏循环
    //这里的每一个system都是异步执行的
    //.add_systems(Update, (person_name,people,people_with_job,people_without_job))//Update在系统运行的每一帧执行一次
    .add_plugins(PeoplePlugin)
    .run();
}

fn _hello_world(){
    println!("hello bevy world ")
}

pub  fn setup(
    mut commands:Commands,
     //Commands 是bevy为我们提供的一个组件，
     //可以使用它来生成实体，取消实体，向实体添加组件，以及从“游戏世界”(系统)中的实体中删除组件
){
    //先实体中添加组件(这里是Person组件),可以使用元组向同一个实体中添加多个组件
    //commands.spawn()函数用于创建实体，并返回一个对实体操作的命令(句柄)。
    commands.spawn(Person{
        name:"Alex".to_string()
    });
    commands.spawn((
        Person{
            name:"Hack".to_string()
        },
        Employed{
            job:Job::Doctor,
        },
    ));
    commands.spawn((
        Person{
            name:"Bob".to_string()
        },
        Employed{
            job:Job::FireFighter,
        },
    ));
    println!("setup called ...")
}

#[derive(Debug,Component)]
pub struct Person{
    pub name:String
}

#[derive(Component)]
pub struct  Employed{
    pub job:Job
}

#[derive(Debug)]
pub enum Job{
    Doctor,
    FireFighter,
}

//ECS就像一个世界，我们所有的数据都在一个世界中，当我们需要获取数据时只需通过Query附件条件查询即可
pub fn person_name(
    person_query:Query<&Person>,//此处参数的意思是，从世界汇总获取所有Person组件的引用。
){
    for person in person_query.iter(){
        println!("person_name:{:?}",person)
    }
}

//使用过滤器
pub fn people_with_job(person_query:Query<&Person,With<Employed>>){
    for person in person_query.iter(){
        println!("people_with_job:{:?}",person)
    }
}
pub fn people_without_job(person_query:Query<&Person,Without<Employed>>){
    for person in person_query.iter(){
        println!("people_without_job:{:?}",person)
    }
}
pub fn people(person_query:Query<(&Person,&Employed)>){
    for (person,employed) in person_query.iter(){
        let job_name = match employed.job {
            Job::Doctor => "Doctor",
            Job::FireFighter => "FireFighter",
        };
        println!("people:{} is a {}",person.name,job_name)
    }
}

//创建自定义插件
pub struct PeoplePlugin;
//实现bevy的插件trait
impl Plugin for PeoplePlugin{
    fn build(&self, app: &mut App) {
        //可以将上边实现的“系统”注册到我们的插件中
        app.add_systems(
            Update, 
            (person_name,people,people_with_job,people_without_job)
        );
    

    }
}
```

```rust

use bevy::{prelude::*, window::PrimaryWindow};
fn main() {
    App::new()
    .add_plugins(DefaultPlugins)
    .add_systems(Startup, (spawn_camera,spawn_player))
    .run();
}

#[derive(Component)]
pub struct Player{}
//bevy为我们创建了一个带有窗口和主窗口组件的实体,以及资产服务资源
pub fn spawn_player( //定义一个名为“spawn_player”的系统
    mut commands:Commands,
    window_query:Query<&Window,With<PrimaryWindow>>,//需要获取“窗口”，以便获取其高度和宽度的信息
    asset_server:Res<AssetServer>,
    /*
    bevy 中的资源是一个独特且可以全局访问的资源，主要用于数据也可以用于功能
    bevy中存在很多系统资源，在我们的“世界”中可以将资源作为“系统”的参数
        Res<T> 只读
        ResMut<T> //可变
    注意：资源不能也不该用作查询参数，他们在我们的“系统中”是它自己的参数--即资源是“系统”参数的一部分
    */
  
){
    //获取窗口的引用
    let window = window_query.get_single().unwrap();
    commands.spawn((
        SpriteBundle{//精灵包 是一个捆绑包，里边包含了一些组件
            //transform 变换组件
            transform:Transform::from_xyz(window.width()/2.0, window.height()/2.0, 0.0),
            //texture ==> 图片句柄
            texture:asset_server.load("sprites/star.png"),
            ..default()
        },
        Player{},
    ));
}

//定义一个生成“相机”的系统
pub fn spawn_camera(
    mut commands:Commands,
    window_query:Query<&Window,With<PrimaryWindow>>,
){
    let window = window_query.get_single().unwrap();
    //创建一个实体并添加一个2d的相机组件
    commands.spawn(
        Camera2dBundle {//使用2D相机精灵包
        //设置相机位置
        transform:Transform::from_xyz(window.width()/2.0, window.height()/2.0, 0.0),//即 居于窗口中间
        //在bevy中坐标的位置坐下角是原点(0,0,0)
        //将基于的组件设置为默认值
        ..default()
    });
}
```
