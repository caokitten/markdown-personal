# net.minecraft

## world

### BlockStateProperties

net.minecraft.world.level.block.state.properties.BlockStateProperties

**提供方块状态属性的预定义常量,描述方块状态**

常量

- BooleanProperty类型
  - POWERED:被激活
  - OPEN:被打开
  - LIT:被点亮
  - WATERLOGGED:含水
  - UP,DOWN,NORTH,SOUTH,WEST,EAST:方向
- IntegerProperty类型
  - POWER:信号强度(0-15)
  - AGE_*:生长阶段
  - NOTE:音符盒(0-24)
- EnumProperty类型
  - FACING:包含(UP,DOWN,NORTH,SOUTH,WEST,EAST),朝向
  - HORIZONTAL_FACING:包含(NORTH,EAST,SOUTH,WEST),水平朝向
  - *_REDSTONE:红石粉连接方向
  - SLAB_TYPE:台阶类型

## core

### BlockPos

net.minecraft.core

方块坐标封装

方法

- 方向移动
  - above(),below(),north(),...参数表示移动n格
  - relative()沿指定方向移动
- 坐标压缩
  - 将三个int压缩为一个long
  - getX()...提取指定坐标值
- 坐标运算
  - offset:坐标偏移
  - subtract:坐标减算
  - multiply:坐标乘算
  - min
  - max
- 旋转:rotate
- 区域迭代
  - betweenClosed...(start,end)支持foreach,stream
  - withinManhattan曼哈顿遍历
  - spiralAround螺旋遍历
- 设置Y:atY
- 叉积:cross

