### An open-source advanced chat addon for users and developers.
Modular, fully customizable and developer-friendly. EasyChat provides a vast amount of features for both users and developers.

### Contributing
Any contributions are welcome, please follow the naming conventions already present in the source code.

### Markup / Item-Tag Implementation（标记效果实现说明）
- EasyChat 的“标记效果”本质上是 ChatHUD 的 **Part（组件）系统**。核心入口在 `lua/easychat/chathud.lua` 的 `RegisterPart`、`CreateComponent`、`PushPartComponent`。
- 聊天字符串会先被 `NormalizeString` 处理，再由 `PushString` 按 `<tag=value>` 解析成组件；组件在绘制时通过 `Draw(ctx)`、`PreTextDraw`、`PostTextDraw` 修改颜色、位移、旋转等渲染状态。
- 默认和扩展示例可直接参考：
  - `lua/easychat/chathud.lua`（基础 tag：`color`、`font`、`stop` 等）
  - `lua/easychat/modules/extra_tags.lua`（额外效果：`c`、`flash`、`hsv`、`scale`、`rotate` 等）

如果你要做一个简易自定义效果，最小实现如下（示例：`<pulse=速度,r,g,b>`）：

```lua
local chathud = EasyChat.ChatHUD
local math_sin = math.sin

local pulse_part = {
	OkInNicks = false,
	Usage = "<pulse=speed,r,g,b>",
	Examples = {
		"<pulse=3,255,64,64>Pulsing text"
	}
}

function pulse_part:Ctor(str)
	local args = str:Split(",")
	self.Speed = math.Clamp(tonumber(args[1]) or 3, 0.1, 20)
	self.TargetColor = Color(
		tonumber(args[2]) or 255,
		tonumber(args[3]) or 64,
		tonumber(args[4]) or 64
	)
	self.Color = Color(self.TargetColor.r, self.TargetColor.g, self.TargetColor.b)
	return self
end

function pulse_part:Draw(ctx)
	local coef = (math_sin(CurTime() * self.Speed) + 1) * 0.5
	self.Color.r = self.TargetColor.r * coef
	self.Color.g = self.TargetColor.g * coef
	self.Color.b = self.TargetColor.b * coef
	ctx:UpdateColor(self.Color)
end

chathud:RegisterPart("pulse", pulse_part)
```

扩展建议：
- 只做颜色/透明度变化：实现 `Ctor + Draw` 就够用（最简单、性能也好）。
- 需要在文字前后绘制背景/描边：用 `ctx:PushPreTextDraw(self)` / `ctx:PushPostTextDraw(self)`。
- 需要允许用户开关：`RegisterPart` 后会自动生成 `easychat_tag_<name>` 的客户端 cvar（除黑名单 tag 外）。

### Details
- The Lua editor which was previously part of this repo has been moved [here](https://github.com/Earu/Lua-Code-Editor).

![chatbox](https://i.imgur.com/vKlszY6.png)
![chathud](https://i.imgur.com/x354846.gif)


#### __*List of URLs & APIs used*__

Internals:
- https://raw.githubusercontent.com/Earu/EasyChat/master/external_data/transliteration_lookup.json *(character lookup)*
- https://raw.githubusercontent.com/Earu/EasyChat/master/external_data/steam_emoticons.txt *(steam emotes)*
- https://raw.githubusercontent.com/Earu/EasyChat/master/external_data/twemojis.json *(twemoji emotes)*
- https://raw.githubusercontent.com/Earu/EasyChat/master/external_data/twemojis.txt.lzma *(twemoji emotes)*

Unlikely to go down:
- http://steamcommunity-a.akamaihd.net/economy/emoticonhover/ *(steam emotes)*
- https://steamcommunity.com/profiles/%s *(menus opening steam profiles)*
- https://api.imgur.com/3/image.json *(pasting images in the chat)*
- https://fonts.googleapis.com/css2?family=Roboto:wght@400&display=swap *(font)*
- https://twemoji.maxcdn.com/v/latest/72x72/%s.png *(twemoji emotes)*

Uncertain:
- https://api.betterttv.net/3/emotes/shared/top *(BTTV emotes)*
- https://cdn.betterttv.net/emote/%s/3x *(BTTV emotes)*
- https://sprays.xerasin.com/legacy/get *(animated emotes)*
- https://api.frankerfacez.com/v1/set/global *(FFZ emotes)*
