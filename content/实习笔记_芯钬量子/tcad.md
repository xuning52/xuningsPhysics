# tcad
官方教程文档
https://www.gmpt.com.cn/documents/nuwa-software-introduce/device_simu_intro

我的思路：基于 Baseline 模型进行二次开发与结构优化
使用Nuwa TCAD软件仿真超结VDMOS器件
[微电子器件 | 产品与应用 | 芯钬量子](https://www.gmpt.com.cn/documents/micro-device/microele_si_power_VDMOS)
我在2026/7/13 15:48向官方发邮件，询问是否能给拿到一份VDMOS仿真源代码。

nuwa tcad输出什么格式？



# Si-VDMOS仿真的尝试
## 一、 需要提前实例化的工艺仿真材料

|**材料名称**|**软件内对应 Alias**|**作用说明**|
|---|---|---|
|**单晶硅**|`Si`|衬底和**单次** $\text{N}$ 型外延层的主体材料。|
|**二氧化硅**|`SiO2`|场氧化层、栅氧化层、BPSG 钝化隔离层。|
|**多晶硅**|`Poly`|用于制作控制栅电极。|
|**金属铝**|`Al`|源极、栅极前段金属化，以及背面漏极金属化。|
|**光刻胶**|`Photores`|用于光刻掩膜以定义注入窗口。|

💡 **掺杂剂准备**：

- **$\text{N}$ 型掺杂剂**：磷（Phosphorus）或砷（Arsenic），用于衬底、**单次 $\text{N}$ 型外延漂移区**和 $\text{N}^+$ 源区。
    
- **$\text{P}$ 型掺杂剂**：硼（Boron），用于 Pbody（P型基区）和 $\text{P}^+$ 欧姆接触区 **（无需用于 P 柱）**。

- **实例化 `SiO2_Poly` 界面**：
    
    - 在下方的列表中找到第 9 行：**`SiO2_Poly`**。
        
    - 点击这一行最右侧对应的 **`Inst.`**（实例化）按钮。
        
    - 在弹出的参数窗口中直接点击 **`Save`** 并返回。
        
- **实例化 `SiO2_Si` 界面**：
    
    - 在下方列表中继续往下拉（或者在右上角的 Search 框里搜一下 `SiO2_Si`）。
        
    - 找到 **`SiO2_Si`**（或者 `Si_SiO2`）这一行，同样点击最右侧的 **`Inst.`** 按钮。
        
    - 直接点击 **`Save`** 并返回。
    

## 二、 构建 2D 结构的详细步骤

由于 Si-VDMOS 的漂移区是均匀掺杂的，我们**不需要多次外延与注入循环**，直接一次性生长出 $\text{N}$ 型漂移区即可。

### 阶段 1：漏极衬底与单个外延漂移区构建

1. **Substrate (定义衬底)**
    
    - 新建衬底，材料选 `Si`，掺杂选择 `Phosphorus`（高浓度，例如 $1\times10^{19}\text{ cm}^{-3}$ 以上），作为 $\text{N}^+$ 漏极衬底。
        
2. **01_Epi_Deposit (单次 N 外延生长)**
    
    - **修改点：** 删去原方案中复杂的 Epi_1~6 循环、牺牲氧化、P柱注入和去胶步骤。
        
    - **操作：** 使用一次 `Deposition/Epitaxy` 沉积 `Si`，进行轻掺杂 `Phosphorus`。
        
    - 厚度与浓度设计参考：对于 600V 器件，外延层厚度一般在 $35\sim45\ \mu\text{m}$，掺杂浓度在 $1\times10^{14}\sim2\times10^{15}\text{ cm}^{-3}$ 左右。
        

### 阶段 2：Pbody（P型基区）结构构建

3. **02_Pbody_Implant (Pbody 注入)**
    
    - **修改点：** 调整步骤顺序，将 Pbody 注入移至栅极光刻前，或者采用自对准工艺。
        
    - **操作：** 使用 `Implantation` 注入 `Boron`，用于形成 VDMOS 的 $\text{P}$ 型基区（沟道承载区）。
        
4. **03_Pbody_Activation (Pbody 高温推结)**
    
    - 运行 `Diffusion` 扩散工艺进行退火推结，使 $\text{P}$ 型基区的结深达到设计要求（通常在 $2\sim3\ \mu\text{m}$ 左右）。
        

### 阶段 3：控制栅极与源区构建（VDMOS 核心）

5. **04_Gate_Oxide_Deposit (栅氧化层生长)**
    
    - 热生长或沉积一层超薄的二氧化硅 `SiO2` 作为栅氧化层。
        
6. **05_Poly_Gate_Deposit (多晶硅栅沉积)**
    
    - 使用 `Deposition/Epitaxy` 沉积多晶硅 `Poly`，并进行高浓度 $\text{N}$ 型掺杂。
        
7. **06_Gate_Etch (栅极光刻与刻蚀)**
    
    - 使用 `Etching/Polishing` 刻蚀多晶硅和栅氧化层，在元胞中心开出源区注入窗口，形成 VDMOS 标志性的**平面多晶硅栅极**。
        
8. **07_NSource_Implant (N+ 源区注入)**
    
    - 以多晶硅栅为掩膜（自对准），在 Pbody 内部注入高浓度的 `Arsenic`（砷），形成 $\text{N}^+$ 源区。然后进行短时间退火激活。
        

### 阶段 4：钝化与电极引出

9. **08_BPSG_Deposit (介质钝化)**
    
    - 沉积 BPSG 钝化层，用于隔离栅极与源极金属。
        
10. **09_Contact_Etch (源极接触孔刻蚀)**
    
    - 刻蚀接触孔。注意：刻蚀需穿透 $\text{N}^+$ 源区，稍微深入硅表面一点，确保金属能同时接触到 $\text{N}^+$ 源区和 Pbody 的中心。
        
11. **10_PPlus_Implant (P+ 接触注入)**
    
    - 在接触孔底部注入高浓度的 `Boron`，形成 $\text{P}^+$ 接触区，以实现源电极与 Pbody 的良好欧姆接触，并有效**短接 $\text{N}^+$ 源区与 Pbody** 以抑制寄生三极管。
        
12. **11_Metal_Al (正面金属化)**
    
    - 全表面沉积金属 `Al`，刻蚀出源极金属（Source Metal）和栅极引出金属。
        
13. **12_Backside_Metal (背面减薄与金属化)**
    
    - 对衬底底部进行减薄，沉积背面金属 `Al`（或金等组合金属），引出漏极（Drain）。

| **工艺阶段** | **仿真步骤**                | **材质 / 杂质**                            | **关键物理参数设计值**                                                                     | **作用与物理设计考量**                                 |
| -------- | ----------------------- | -------------------------------------- | --------------------------------------------------------------------------------- | --------------------------------------------- |
| **阶段 1** | **Substrate**           | $\text{Si}$ / $\text{Phosphorus}$      | 厚度：$100\ \mu\text{m}$ (仿真推荐)<br><br>  <br><br>浓度：$2\times 10^{19}\text{ cm}^{-3}$ | $\text{N}^+$ 漏极衬底，减小背面接触电阻                    |
|          | **01_Epi_Deposit**      | $\text{Si}$ / $\text{Phosphorus}$      | 厚度：$42\ \mu\text{m}$<br><br>  <br><br>浓度：$1.2\times 10^{14}\text{ cm}^{-3}$       | $\text{N}^-$ 漂移区。确保未耗尽时阻断电压 $\ge 600\text{V}$ |
| **阶段 2** | **02_Pbody_Implant**    | $\text{Boron}$                         | 能量：$80\text{ keV}$<br><br>  <br><br>剂量：$1.5\times 10^{13}\text{ cm}^{-2}$         | 注入形成 $\text{P}$ 型基区，决定沟道掺杂                    |
|          | **03_Pbody_Activation** | 扩散 / 退火                                | 温度：$1100^\circ\text{C}$<br><br>  <br><br>时间：$150\text{ min}$                      | 推结使结深达 $2.5\ \mu\text{m}$，控制横向沟道长度            |
| **阶段 3** | **04_Gate_Oxide**       | $\text{SiO}_2$                         | 厚度：$80\text{ nm}$ (干氧生长)                                                          | 栅氧化层。决定跨导与 $V_{th}$ 窗口                        |
|          | **05_Poly_Gate**        | $\text{Poly-Si}$ / $\text{Phosphorus}$ | 厚度：$0.4\ \mu\text{m}$<br><br>  <br><br>浓度：$1\times 10^{20}\text{ cm}^{-3}$        | 高掺杂多晶硅栅，确保功函数稳定                               |
|          | **06_Gate_Etch**        | 刻蚀                                     | 窗口宽度：约 $4\sim 6\ \mu\text{m}$                                                     | 元胞中心开窗，确定后续注入位置                               |
|          | **07_NSource_Implant**  | $\text{Arsenic}$                       | 能量：$50\text{ keV}$<br><br>  <br><br>剂量：$5\times 10^{15}\text{ cm}^{-2}$           | 自对准注入。形成浅且高浓度的 $\text{N}^+$ 源区                |
| **阶段 4** | **08_BPSG_Deposit**     | $\text{BPSG}$                          | 厚度：$0.8\ \mu\text{m}$                                                             | 钝化介质层，隔离栅源金属                                  |
|          | **09_Contact_Etch**     | 硅刻蚀                                    | 深度：穿透源区并深入硅 $0.15\ \mu\text{m}$                                                   | 露出 $\text{N}^+$ 与 Pbody，准备进行短接                |
|          | **10_PPlus_Implant**    | $\text{Boron}$                         | 能量：$30\text{ keV}$<br><br>  <br><br>剂量：$2\times 10^{15}\text{ cm}^{-2}$           | $\text{P}^+$ 接触注入。抑制寄生三极管开启                   |
|          | **11_Metal_Al**         | $\text{Aluminum}$                      | 厚度：$3\ \mu\text{m}$ (正面)                                                          | 铺满接触孔，形成源电极与栅电极引出                             |
|          | **12_Backside_Metal**   | $\text{Aluminum}$                      | 厚度：$1\ \mu\text{m}$ (背面)                                                          | 漏极电极                                          |
