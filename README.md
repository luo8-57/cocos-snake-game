# 🐍 Cocos Creator 2D 贪吃蛇小游戏

基于 **Cocos Creator 3.8** 开发的经典 2D 贪吃蛇小游戏，完整实现了核心玩法与基础游戏逻辑，是学习 2D 游戏开发的练手项目。

---

## 📋 项目说明（版权声明）

本项目为个人学习练手项目，核心逻辑参考了 **siki学院《三天学会开发贪吃蛇》教程** 实现，仅用于个人学习与技术展示，无任何商业用途。

---

## 🛠️ 技术栈

| 技术 | 版本/说明 |
|------|-----------|
| **游戏引擎** | Cocos Creator 3.8 |
| **编程语言** | TypeScript |
| **核心特性** | 组件化开发、Prefab 预制体复用、2D 碰撞检测、UITransform 坐标管理 |
| **版本控制** | Git |

---

## 🎮 核心功能实现

### 1. 🐍 蛇头控制与移动

- **移动逻辑**：通过 `deltaTime` 控制移动速度，避免帧率波动影响游戏体验
- **随机生成**：游戏启动时自动在安全区域内随机生成蛇头初始位置，避免出界
- **方向控制**：通过虚拟摇杆（Joystick）控制蛇头移动方向
- **核心代码**：通过 `UITransform` 获取画布尺寸，计算随机坐标

```typescript
// 随机生成位置
randomPos() {
    let width = this.node.parent.getComponent(UITransform).contentSize.width - 200;
    let height = this.node.parent.getComponent(UITransform).contentSize.height - 200;
    let x = Math.round(Math.random() * width) - width / 2;
    let y = Math.round(Math.random() * height) - height / 2;
    return v3(x, y, 0);
}
```

### 2. 🐛 蛇身跟随与增长

- **数组管理**：用数组 `bodyArray` 管理所有蛇身节点，通过记录蛇头历史位置实现蛇身平滑跟随
- **动态生成**：吃到食物后动态生成蛇身节点，通过预制体复用降低性能消耗
- **跟随算法**：每帧记录蛇头位置，蛇身依次移动到前一个节点的位置

```typescript
// 蛇身跟随移动
moveBody() {
    let headPos = this.node.position; // 保存蛇头移动前的位置
    for (let i = this.bodyArray.length - 2; i >= 0; i--) {
        this.bodyArray[i + 1].position = this.bodyArray[i].position;
    }
    this.bodyArray[1].position = headPos;
}
```

### 3. 🍎 食物生成与碰撞检测

- **随机生成**：食物在画布安全区域内随机生成，避免生成在蛇身上
- **随机颜色**：食物颜色随机生成，增加视觉趣味性
- **三类碰撞检测**：
  - ✅ **蛇头与食物碰撞**：蛇身增长 + 食物重新生成 + 播放音效
  - ❌ **蛇头与边界碰撞**：游戏结束 + 播放死亡音效
  - ❌ **蛇头与自身碰撞**：游戏结束 + 播放死亡音效

```typescript
// 碰撞检测处理
onBeginContact(selfCollider: Collider2D, otherCollider: Collider2D) {
    if (otherCollider.group == 4) { // 食物碰撞
        otherCollider.node.removeFromParent();
        this.Score++;
        this.txt_Score.string = "<color=#000000>" + this.Score.toString() + "</color>";
        let newFood = instantiate(this.foodPrefab);
        this.node.parent.addChild(newFood);
        this.getNewBody();
        this.node.getComponent(AudioSource).playOneShot(this.eatSound);
    }
    if (otherCollider.group == 8 || otherCollider.group == 16) { // 边界/自身碰撞
        this.gameOverPanel.active = true;
        this.gameOverPanel.getChildByName("Txt_Score").getComponent(Label).string = "得分:" + this.Score.toString();
        this.node.getComponent(AudioSource).playOneShot(this.DieSound);
        director.pause();
    }
}
```

### 4. 🎯 项目工程化设计

- **组件化开发**：蛇头、蛇身、食物分别拆分为独立组件，代码解耦易维护
- **编辑器暴露参数**：核心参数（蛇身数量、移动速度、身体间距）支持在编辑器中快速调试
- **预制体复用**：使用 Prefab 预制体管理重复使用的节点，提高开发效率

---

## 📁 项目结构

```
cocos-snake-game/
├── .creator/                          # Cocos Creator 编辑器配置
│   ├── asset-template/                # 资产模板
│   └── default-meta.json             # 默认元数据配置
├── assets/                            # 游戏资源目录
│   ├── prefabs/                       # 预制体
│   │   ├── Body.prefab               # 蛇身预制体
│   │   ├── Food.prefab               # 食物预制体
│   │   └── Head.prefab               # 蛇头预制体
│   ├── res/                           # 资源文件
│   │   ├── audios/                    # 音频资源
│   │   │   ├── BGM.wav               # 背景音乐
│   │   │   ├── Die.wav               # 死亡音效
│   │   │   └── Eat.wav               # 进食音效
│   │   ├── textures/                  # 纹理图片
│   │   │   ├── Bg.png                # 背景图片
│   │   │   ├── Body.png              # 蛇身图片
│   │   │   └── icecream-*.png        # 食物图片（冰淇淋系列）
│   │   └── 免费2D卡通UI素材包/        # UI 素材包
│   │       └── ui-assets/            # 按钮、光标、符号等 UI 元素
│   └── scripts/                       # TypeScript 脚本
│       ├── Food.ts                    # 食物组件
│       ├── Head.ts                    # 蛇头组件（核心逻辑）
│       └── Joystick.ts               # 虚拟摇杆组件
├── profiles/                          # 配置文件
├── .gitignore                         # Git 忽略规则
├── package.json                       # 项目配置
├── tsconfig.json                      # TypeScript 配置
└── README.md                          # 项目说明文档
```

---

## 🎮 游戏玩法

### 基本操作
- **移动控制**：使用虚拟摇杆控制蛇头移动方向
- **进食**：蛇头碰到食物（冰淇淋）后，蛇身增长，得分增加
- **游戏结束**：蛇头碰到边界或自身时，游戏结束

### 游戏目标
- 尽可能多地进食食物，获得高分
- 避免碰到边界和自身

---

## 🚀 本地运行步骤

### 1. 克隆仓库到本地

```bash
git clone https://github.com/luo8-57/cocos-snake-game.git
```

### 2. 安装 Cocos Creator

- 下载并安装 [Cocos Creator 3.8](https://www.cocos.com/creator)
- 确保安装了 TypeScript 支持

### 3. 打开项目

1. 启动 Cocos Creator
2. 选择 "打开项目"
3. 选择克隆下来的 `cocos-snake-game` 文件夹
4. 等待项目加载完成

### 4. 运行游戏

1. 在 Cocos Creator 中打开场景文件（通常在 `assets/scenes/` 目录下）
2. 点击顶部工具栏的 "预览" 按钮（或按 `Ctrl+P`）
3. 选择浏览器或模拟器运行
4. 开始游戏！

---

## 🔧 核心参数配置

在 Cocos Creator 编辑器中，可以调整以下参数：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `bodyNum` | 初始蛇身数量 | 2 |
| `bodyDistance` | 蛇身之间的距离 | 50 |
| `speed` | 蛇移动速度 | 200 |
| `eatSound` | 进食音效 | Eat.wav |
| `DieSound` | 死亡音效 | Die.wav |

---

## 📚 学习要点

### TypeScript 在 Cocos Creator 中的应用

1. **装饰器使用**
   - `@ccclass('ClassName')`：注册类为 Cocos 组件
   - `@property(Type)`：暴露属性到编辑器

2. **组件生命周期**
   - `onLoad()`：组件加载时调用
   - `start()`：组件第一次激活前调用
   - `update(deltaTime)`：每帧调用

3. **碰撞检测**
   - 使用 `CircleCollider2D` 组件
   - 监听 `Contact2DType.BEGIN_CONTACT` 事件

4. **预制体实例化**
   - 使用 `instantiate()` 方法动态创建节点
   - 通过预制体复用提高性能

---

## 🎨 资源说明

### 音频资源
- **BGM.wav**：游戏背景音乐
- **Eat.wav**：进食音效
- **Die.wav**：死亡音效

### 图片资源
- **Bg.png**：游戏背景
- **Body.png**：蛇身纹理
- **icecream-*.png**：食物纹理（10 种不同颜色的冰淇淋）

### UI 素材
- 包含丰富的 2D 卡通 UI 元素
- 按钮、滑块、图标、符号等
- 可用于扩展游戏 UI

---

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建你的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交你的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

---

## 📄 许可证

本项目仅供学习交流使用，请勿用于商业用途。

---

## 🙏 致谢

- [Cocos Creator](https://www.cocos.com/creator) - 游戏引擎
- [siki学院](https://www.sikiedu.com/) - 教程参考
- [免费2D卡通UI素材包](https://assetstore.unity.com/) - UI 资源

---

## 📞 联系方式

如有问题或建议，欢迎通过以下方式联系：

- GitHub Issues：[提交问题](https://github.com/luo8-57/cocos-snake-game/issues)
- 邮箱：your-email@example.com

---

⭐ 如果这个项目对你有帮助，请给个 Star 支持一下！
