# 3D 设计学习 + Tripo 吉祥物转 3D 流水线备战笔记

> 研究时间：2026-10-17 11:50（北京）。pan 刚指示"接着自主学习设计水平，包括平面设计和3D设计"（第四轮进行中）。
> 本笔记是 3D 轨道的备战材料：Tripo API 集成已调研到可直接开跑的程度，只差 pan 的 API key；
> Blender 学习路线按他的生意（广告标识/包装/吉祥物）裁剪，不走通用游戏建模弯路。

## 一、3D 对 pan 的生意价值（先说钱）

1. **标识效果图渲染**：客户要看"装上墙什么样"，现在靠 PS 贴图容易"穿模"；Blender 里建个门头/精神堡垒粗模+打光渲染，效果图一次过稿率会高很多。
2. **包装打样**：包装盒案例是 pan 点名要重设计的；Blender 折纸盒 UV 展开+渲染，比平面拼贴可信一个量级。
3. **吉祥物 3D 化**：网站吉祥物是 Q 版人物，pan 要求"助理头像本人动起来（招手/走路/跳）"——这是 Tripo 图生 3D + 自动骨骼绑定的标准活。
4. **菜单场景图**：辉味菜单的 44 张占位图色温混乱；以后菜品用 3D 场景统一打光渲染，一套感直接拉满。

## 二、Tripo 图生 3D 流水线（已备战，待 pan 给 API key）

### 注册与费用（国内站）
- 国内站：https://developers.tripo3d.com ——中文界面、人民币计价，账号与国际站不互通（之前已跟 pan 纠正过一次）。
- 登录：手机号 + 短信验证码（没注册过会自动注册）。
- 计费：1 积分 = ¥0.07；3D 生成单次约 10+ 积分起（≈ ¥0.7~几块钱，取决于纹理档位），自动绑定/动画约 25 积分起。先小额充几十块跑一张验证。
- pan 的动作：注册 → 在控制台创建 API key → 通过安全链接（不是聊天）发给 Muse。

### API 集成要点（V3，V2 已于 2026-10-01 退役，新集成必须走 V3）
- 认证：`Authorization: Bearer <API_KEY>`；任务异步：POST 建任务 → 轮询 `/task/{task_id}` → 取 `output.pbr_model`（带 PBR 贴图的 GLB）下载。
- 图片先上传换 `image_token`（POST /upload），再提交 `image_to_model` 任务；也支持直接传公网 `image_url`。
- 吉祥物推荐参数：
  - `texture: true, pbr: true`（要贴图就得开；关掉两个才便宜但没用）
  - `texture_quality: "standard"`（先跑通再试 detailed/extreme，extreme 是 8K 纹理更贵）
  - `face_limit: 50000`，`quad: true`（四边面拓扑，后续绑定做动画更干净）
  - `delight: true`（去掉原图 baked-in 光照，纹理更干净）
  - 不要设 `export_orientation`（文档建议：后续还要做绑定/动画时别动朝向参数）
- 输入图要求（决定成败的一半）：**单主体、干净白底、3/4 正面微俯视角、哑光质感**。pan 选的是 3 张季节招手版吉祥物图——注意：招手姿势的图直接转 3D 后做动画较难；专业路线是**用中性站姿图生成 + Tripo 自动绑定（auto-rig）+ 在 Blender 里摆招手动画**。建议 key 到手先跑 1 张验证，看效果再决定 3 张全跑还是换中性姿势图。
- 输出：GLB（网页直接用）/ FBX（进 Blender 细化）。Tripo 还有 Blender 插件，DCC 流程可打通。
- 待确认（key 到手登录国内站文档核对一次）：国内站 API host 与鉴权是否与国际站一致；国内站文档在 developers.tripo3d.com 控制台内。

### key 到手后的开跑清单（届时直接执行）
1. 准备 3 张季节吉祥物图（中性底版优先，如无则用招手版先验证 1 张）。
2. 调国内站 API：upload → image_to_model（上表参数）→ 轮询 → 下载 GLB。
3. Blender 导入检查：破面/贴图错位 → 决定修补或重跑（换 texture_seed 可换纹理不换几何）。
4. 需要动画：走 Tripo rigging/retarget 或 Blender 手动绑骨 → 导出 GLB → 替换网站 WebP（先 A/B 验证再换）。

## 三、Blender 学习路线（为广告设计师裁剪，4 周）

> 目标不是"会建模"，是"能为接单提速"。Blender 免费，官网 blender.org 下载 4.x。

- **第 1 周 · 界面与硬表面基础**：跟着 Blender Guru 的 Beginner 系列（英文但语速慢、可开字幕）走完界面→基础建模→材质→打光→渲染全流程；练习：建一个精神堡垒粗模并渲染效果图。
- **第 2 周 · 标识/包装专项**：文本转 3D 发光字（解决"笔画<15mm 别做树脂字"这类工艺直觉的可视化）、纸盒 UV 展开与折叠、HDRI 打光（polyhaven.com 免费 HDR）；练习：给包装盒案例出一张 3D 打样渲染。
- **第 3 周 · 吉祥物与动画**：Sculpt 模式微调 Tripo 生成的网格、自动权重绑骨、关键帧做招手/走路循环、导出 GLB；练习：让 Q 版人物在网站上招手。
- **第 4 周 · 合成与交付**：Compositor 后期调色、Eevee（快）vs Cycles（真）渲染取舍、输出规范（网站用 WebP/GLB、印刷用高分辨率 PNG）。
- 中文资源：BlenderCN 社区、intranetgirl（偏平面设计审美）、Graphic Income（Illustrator+Blender 联动、模型变现思路——和 pan 的"学商插拓宽收入"目标同构）。

## 四、与现有工作的衔接

- 吉祥物 3D：接 MEMORY 里"等 pan 的 Tripo API key"那条线，本笔记让 key 到手后零研究直接开跑。
- 包装盒案例重设计（pan 点名不满意）：Blender 打样渲染是重设计交付的加分项，可在候选定稿后加一张 3D 效果图。
- 本笔记放入 `~/workspace/design-study/notes/3d-tripo-pipeline.md`，与 9 份笔记 + 3 份实战并列；第四轮研究员可直接引用第二、三节。

## 来源
- Tripo API V3 参数与退役信息：https://www.anygen.io/showcase/tripo-ai/index.html（V3 路由、webhook、计费档位）、https://developers.tripo3d.ai/en/docs/generation-text-to-model/p（texture_quality/texture_version/delight/auto_size/export_orientation 参数说明）、https://www.pixazo.ai/models/tripo3d（face_limit/quad/pbr 参数表）
- Blender 入门路线：https://www.coohom.com/article/how-to-use-blender-to-create-3d-models-a-beginner's-guide（primitives→edit→modifiers→materials 工作流）、https://www.lemon8-app.com/experience/สอน-blender（Blender Guru/ intranetgirl/Graphic Income 频道推荐）
- pan 的原话与待办：source:251（"接着自主学习设计水平，包括平面设计和3d设计"）、MEMORY（Tripo 国内站、等 API key、吉祥物动画要求）
