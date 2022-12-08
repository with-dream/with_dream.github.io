#### 一、混合
将准备进入帧缓冲的片元(源片元)与帧缓冲中的片元(目标片元)按照比例加权计算出最终片元的颜色

混合的比例为源/目标因子  有四个颜色通道 所以混合因子有四个值

```glsl
//混合方法
//sfactor:源混合因子   dfactor:目标混合因子
glBlendFunc(GLenum sfactor, GLenum dfactor)
//srcRGB:源混合的RGB因子   srcAlpha:源混合的Alpha因子
glBlendFuncSeparate(GLenum srcRGB, GLenum dstRGB, GLenum srcAlpha, GLenum dstAlpha)
```