# 问题分析与修复方案

## 发现的问题

### ThreeScene.vue 文件
1. **拼写错误**：
   - 第 60 行：`WebGLRednerer` 拼写错误，应为 `WebGLRenderer`
   - 第 93 行：`AmbienetLight` 拼写错误，应为 `AmbientLight`
   - 第 94 行：重复添加了 `ambientLight`，应添加 `directionalLight`
   - 第 119 行：`chatShadow` 拼写错误，应为 `castShadow`

2. **逻辑问题**：
   - 第 75 行：`OrbitControls` 的 `minDistance` 设置为 50，导致初始视角无法看到场景物体（相机初始位置为 (0,5,15)）
   - 第 176-195 行：`loadGLTFModel` 方法是异步的，但调用时未使用 `await`
   - 第 197-210 行：`setupPostProcessing` 中使用了 `EffectComposer`，但在 `beforeDestroy` 中未释放相关资源
   - 第 242-250 行：`onResize` 方法直接使用 `window.innerWidth` 和 `window.innerHeight`，未使用组件的 `width` 和 `height` 属性

3. **依赖问题**：
   - 项目依赖了 `three-gltf-loader` 和 `three-orbit-controls`，但代码中使用的是 three/examples/jsm 中的版本，这些依赖可能多余

### App.vue 文件
1. **功能问题**：
   - 第 10-11 行：`toggleEffect` 和 `changeColor` 方法只是打印日志，没有实际功能

2. **CSS 问题**：
   - 第 54 行：`background-filter` 不是标准 CSS 属性，应为 `backdrop-filter`

## 修复方案

### ThreeScene.vue 修复
1. 修正所有拼写错误
2. 调整 `OrbitControls` 的 `minDistance` 值为合理范围（如 5）
3. 优化 `loadGLTFModel` 方法的调用方式
4. 在 `beforeDestroy` 中添加 `composer` 的释放逻辑
5. 修改 `onResize` 方法，使用组件的 `width` 和 `height` 属性
6. 移除多余的依赖

### App.vue 修复
1. 实现 `toggleEffect` 和 `changeColor` 方法的实际功能
2. 修正 CSS 属性名称

### 其他优化
1. 考虑使用 Vue 3 的组合式 API
2. 添加 TypeScript 类型定义
3. 增强错误处理和边界情况处理

## 实施步骤
1. 修复 ThreeScene.vue 中的拼写错误和逻辑问题
2. 修复 App.vue 中的功能和 CSS 问题
3. 清理多余的依赖
4. 测试修复后的代码运行情况