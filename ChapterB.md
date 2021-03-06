
代码速查手册（B区）
==
#技能索引
[八阵](#八阵)、[霸刀](#霸刀)、[霸王](#霸王)、[拜印](#拜印)、[豹变](#豹变)、[暴虐](#暴虐)、[悲歌](#悲歌)、[北伐](#北伐)、[崩坏](#崩坏)、[笔伐](#笔伐)、[闭月](#闭月)、[补益](#补益)、[不屈](#不屈)、[不屈-旧风](#不屈-旧风) 

[返回目录](README.md#目录)
##八阵
**相关武将**：火·诸葛亮  
**描述**：**锁定技，**若你的装备区没有防具牌，视为你装备着【八卦阵】。  
**引用**：LuaBazhen  
**状态**：1217验证通过
```lua
	LuaBazhen = sgs.CreateTriggerSkill{
		name = "LuaBazhen",
		frequency = sgs.Skill_Compulsory,
		events = {sgs.CardAsked},
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			local pattern = data:toStringList()[1]
			if pattern == "jink" then
				if player:askForSkillInvoke("EightDiagram",data) then
					local judge = sgs.JudgeStruct()
					judge.pattern = ".|red"
					judge.good = true
					judge.reason = self:objectName()
					judge.who = player
					judge.play_animation = true
					room:setEmotion(player, "armor/EightDiagram");
					room:judge(judge)
					if judge:isGood() then
						local jink = sgs.Sanguosha:cloneCard("jink", sgs.Card_NoSuit, 0)
						jink:setSkillName(self:objectName())
						room:provide(jink)
						return true
					end
				end
			end
			return false
		end,
		can_trigger = function(self, target)
			if target then
				if target:isAlive() and target:hasSkill(self:objectName()) then
					if not target:getArmor() then
						if target:getMark("Armor_Nullified")==0 and not target:hasFlag("WuqianTarget") then
							if target:getMark("Equips_Nullified_to_Yourself") == 0 then
								local list = target:getTag("Qinggang"):toStringList()
								return #list == 0
							end
						end
					end
				end
			end
			return false
		end
	}
```
[返回索引](#技能索引) 

##霸刀
**相关武将**：智·华雄  
**描述**：当你成为黑色的【杀】目标后，你可以使用一张【杀】  
**引用**：LuaBadao  
**状态**：1217验证通过
```lua
	LuaBadao = sgs.CreateTriggerSkill{
		name = "LuaBadao" ,
		events = {sgs.TargetConfirmed} ,
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			local use = data:toCardUse()
			if use.card:isKindOf("Slash") and use.card:isBlack() and use.to:contains(player) then
				room:askForUseCard(player, "slash", "@askforslash")
			end
			return false
		end
	}
```
[返回索引](#技能索引) 

##霸王
**相关武将**：智·孙策  
**描述**：当你使用的【杀】被【闪】抵消时，你可以与目标角色拼点：若你赢，可以视为你对至多两名角色各使用了一张【杀】（此杀不计入每阶段的使用限制）  
**引用**：LuaBawang  
**状态**：1217验证通过
```lua
	LuaBawangCard = sgs.CreateSkillCard{
		name = "LuaBawangCard" ,
		filter = function(self, targets, to_select)
			if #targets >= 2 then return false end
			return sgs.Self:canSlash(to_select, false)
		end ,
		on_effect = function(self, effect)
			local room = effect.to:getRoom()
			local use = sgs.CardUseStruct()
			local slash = sgs.Sanguosha:cloneCard("slash", sgs.Card_NoSuit, 0)
			slash:setSkillName("LuaBawang")
			use.card = slash
			use.from = effect.from
			use.to:append(effect.to)
			room:useCard(use, false)
		end
	}
	LuaBawangVS = sgs.CreateViewAsSkill{
		name = "LuaBawang" ,
		n = 0,
		view_as = function()
			return LuaBawangCard:clone()
		end ,
		enabled_at_play = function()
			return false
		end ,
		enabled_at_response = function(self, player, pattern)
			return pattern == "@@LuaBawang"
		end
	}
	LuaBawang = sgs.CreateTriggerSkill{
		name = "LuaBawang" ,
		events = {sgs.SlashMissed} ,
		view_as_skill = LuaBawangVS ,
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			local effect = data:toSlashEffect()
			if (not effect.to:isNude()) and (not player:isKongcheng()) and (not effect.to:isKongcheng()) then
				if room:askForSkillInvoke(player, self:objectName(), data) then
					local success = player:pindian(effect.to, self:objectName(), nil)
					if success then
						if player:hasFlag("drank") then
							room:setPlayerFlag(player, "-drank")
						end
						room:askForUseCard(player, "@@bawang", "@bawang")
					end
				end
			end
			return false
		end
	}
```
[返回索引](#技能索引) 


##拜印
**相关武将**：神·司马懿  
**描述**：**觉醒技，**回合开始阶段开始时，若你拥有4枚或更多的“忍”标记，你须减1点体力上限，并获得技能“极略”。  
**引用**：LuaBaiyin  
**状态**：1217验证通过
```lua
	LuaBaiyin = sgs.CreateTriggerSkill{
		name = "LuaBaiyin" ,
		frequency = sgs.Skill_Wake ,
		events = {sgs.EventPhaseStart} ,
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			room:setPlayerMark(player,"LuaBaiyin",1)
			if room:changeMaxHpForAwakenSkill(player) then
				room:acquireSkill(player, "jilve")
			end
			return false
		end ,
		can_trigger = function(self,target)
			return (target and target:isAlive() and target:hasSkill(self:objectName()))
					and (target:getPhase() == sgs.Player_Start)
					and (target:getMark("LuaBaiyin") == 0)
					and (target:getMark("@bear") >= 4)
		end
	}
```
[返回索引](#技能索引) 

##豹变
**相关武将**：SP·夏侯霸  
**描述**：**锁定技，**若你的体力值为3或更少，你视为拥有技能“挑衅”;若你的体力值为2或更少;你视为拥有技能“咆哮”;若你的体力值为1，你视为拥有技能“神速”。  
**引用**：LuaBaobian  
**状态**：1217验证通过
```lua
	function BaobianChange(room, player, hp, skill_name)
		local baobian_skills = player:getTag("LuaBaobianSkills"):toString():split("+")	
		if player:getHp() <= hp then		
			if not table.contains(baobian_skills, skill_name) then			
				room:notifySkillInvoked(player, "baobian")
				if player:getHp() == hp then				
					room:broadcastSkillInvoke("baobian", 4 - hp)
				end			
				room:handleAcquireDetachSkills(player, skill_name)
				table.insert(baobian_skills, skill_name)
			end
		else
	        if table.contains(baobian_skills, skill_name) then			
				room:handleAcquireDetachSkills(player, "-"..skill_name)
				table.removeOne(baobian_skills, skill_name)
			end
		end
		player:setTag("LuaBaobianSkills", sgs.QVariant(table.concat(baobian_skills, "+")))	
	end
	LuaBaobian = sgs.CreateTriggerSkill{
		name = "LuaBaobian" ,
		events = {sgs.TurnStart, sgs.HpChanged, sgs.MaxHpChanged, sgs.EventAcquireSkill, sgs.EventLoseSkill} ,
		frequency = sgs.Skill_Compulsory ,
		on_trigger = function(self, event, player, data)		
			local room = player:getRoom()
			if event == sgs.TurnStart then			
				local xiahouba = room:findPlayerBySkillName(self:objectName())
				if not xiahouba or not xiahouba:isAlive() then return false end
				BaobianChange(room, xiahouba, 1, "shensu")
		        BaobianChange(room, xiahouba, 2, "paoxiao")
		        BaobianChange(room, xiahouba, 3, "tiaoxin")
		    end
			if event == sgs.EventLoseSkill then
				if data:toString() == self:objectName() then
					local baobian_skills = player:getTag("LuaBaobianSkills"):toString():split("+")
					local detachList = {}
					for _, skill_name in ipairs(baobian_skills) do
						table.insert(detachList,"-"..skill_name)
					end
					room:handleAcquireDetachSkills(player, table.concat(detachList,"|"))
					player:setTag("LuaBaobianSkills", sgs.QVariant())
				end
				return false
			elseif event == sgs.EventAcquireSkill then
				if data:toString() ~= self:objectName() then return false end
			end
			if not player:isAlive() or not player:hasSkill(self:objectName(), true) then return false end		
			BaobianChange(room, player, 1, "shensu")
	        BaobianChange(room, player, 2, "paoxiao")
	        BaobianChange(room, player, 3, "tiaoxin")        
			return false
		end ,
		can_trigger = function(self, target)
			return target ~= nil
		end
	}
```
[返回索引](#技能索引) 

##暴虐
**相关武将**：林·董卓  
**描述**：**主公技，**每当其他群雄角色造成一次伤害后，该角色可以进行一次判定，若判定结果为黑桃，你回复1点体力。  
**引用**：LuaBaonve  
**状态**：1217验证通过
```lua
	LuaBaonve = sgs.CreateTriggerSkill{
		name = "LuaBaonve$",
		frequency = sgs.Skill_NotFrequent,
		events = {sgs.Damage, sgs.PreDamageDone},
		on_trigger = function(self, event, player, data)
			local damage = data:toDamage()
			local room = player:getRoom()
			if (event == sgs.PreDamageDone) and damage.from then
				damage.from:setTag("InvokeLuaBaonve", sgs.QVariant(damage.from:getKingdom() == "qun"))
			elseif (event == sgs.Damage) and player:getTag("InvokeLuaBaonve"):toBool() and player:isAlive() then
				local dongzhuos = sgs.SPlayerList()
				for _, p in sgs.qlist(room:getOtherPlayers(player)) do
					if p:hasLordSkill(self:objectName()) then
						dongzhuos:append(p)
					end
				end
				while not dongzhuos:isEmpty() do
					local dongzhuo = room:askForPlayerChosen(player, dongzhuos, self:objectName(), "@baonve-to", true)
					if dongzhuo then
						dongzhuos:removeOne(dongzhuo)
						local judge = sgs.JudgeStruct()
						judge.pattern = ".|spade"
						judge.good = true
						judge.reason = self:objectName()
						judge.who = player
						room:judge(judge)
						if judge:isGood() then
							recover = sgs.RecoverStruct()
							recover.who = player
							room:recover(dongzhuo, recover)
						end
					else
						break
					end
				end
			end
			return false
		end,
		can_trigger = function(self, target)
			return target
		end
	}
```
[返回索引](#技能索引) 

##悲歌
**相关武将**：山·蔡文姬、SP·蔡文姬  
**描述**：每当一名角色受到【杀】造成的一次伤害后，你可以弃置一张牌，令其进行一次判定，判定结果为：红桃 该角色回复1点体力；方块 该角色摸两张牌；梅花 伤害来源弃置两张牌；黑桃 伤害来源将其武将牌翻面。  
**引用**：LuaBeige  
**状态**：1217验证通过
```lua
	LuaBeige = sgs.CreateTriggerSkill{
		name = "LuaBeige",
		frequency = sgs.Skill_NotFrequent,
		events = {sgs.Damaged},
		on_trigger = function(self, event, player, data)
			local damage = data:toDamage()
			local card = damage.card
			if card and card:isKindOf("Slash") then
				local victim = damage.to
				if not victim:isDead() then
					local room = player:getRoom()
					local list = room:findPlayersBySkillName(self:objectName())
					for _,p in sgs.qlist(list) do
						if not p:isNude() then
							if p:askForSkillInvoke(self:objectName(), data) then
								room:askForDiscard(p, self:objectName(), 1, 1, false, true)
								local judge = sgs.JudgeStruct()
								judge.pattern = "."
								judge.good = true
								judge.who = victim
								judge.reason = self:objectName()
								room:judge(judge)
								local suit = judge.card:getSuit()
								local source = damage.from
								if suit == sgs.Card_Spade then
									if source and source:isAlive() then
										source:turnOver()
									end
								elseif suit == sgs.Card_Heart then
									local recover = sgs.RecoverStruct()
									recover.who = p
									room:recover(victim, recover)
								elseif suit == sgs.Card_Club then
									if source and source:isAlive() then
										local count = source:getCardCount(true)
										if count > 2 then
											count = 2
										end
										if count > 0 then
											room:askForDiscard(source, self:objectName(), count, count, false, true)
										end
									end
								elseif suit == sgs.Card_Diamond then
									victim:drawCards(2)
								end
							end
						end
					end
				end
			end
		end,
		can_trigger = function(self, target)
			return target ~= nil
		end
	}
```
[返回索引](#技能索引) 

##北伐
**相关武将**：智·姜维  
**描述**：**锁定技，**当你失去最后的手牌时，视为你对一名其他角色使用了一张【杀】，若不能如此做，则视为你对自己使用了一张【杀】  
**引用**：LuaBeifa  
**状态**：1217验证通过
```lua
	LuaBeifa = sgs.CreateTriggerSkill{
		name = "LuaBeifa" ,
		frequency = sgs.Skill_Compulsory ,
		events = {sgs.BeforeCardsMove, sgs.CardsMoveOneTime} ,
		on_trigger = function(self, event, player, data)
			local move = data:toMoveOneTime()
			if move.from and (move.from:objectName() == player:objectName()) and move.from_places:contains(sgs.Player_PlaceHand) then
				if event == sgs.BeforeCardsMove then
					if player:isKongcheng() then return false end
					for _, id in sgs.qlist(player:handCards()) do
						if not move.card_ids:contains(id) then return false end
					end
					player:addMark(self:objectName())
				else
					local room = player:getRoom()
					if player:getMark(self:objectName()) == 0 then return false end
					player:removeMark(self:objectName())
					local players = sgs.SPlayerList()
					local slash = sgs.Sanguosha:cloneCard("slash", sgs.Card_NoSuit, 0)
					slash:setSkillName(self:objectName())
					for _, _player in sgs.qlist(room:getOtherPlayers(player)) do
						if player:canSlash(_player, slash) then
							players:append(_player)
						end
					end
					local target = nil
					if not players:isEmpty() then
						target = room:askForPlayerChosen(player, players, self:objectName()) --没有处理TarMod
					end
					if (not target) and (not player:isProhibited(player, slash)) then
						target = player
					end
					local use = sgs.CardUseStruct()
					use.card = slash
					use.from = player
					use.to:append(target)
					room:useCard(use)
				end
			end
			return false
		end
	}
```
[返回索引](#技能索引) 


##崩坏
**相关武将**：林·董卓  
**描述**：**锁定技，**回合结束阶段开始时，若你不是当前的体力值最少的角色之一，你须失去1点体力或减1点体力上限。  
**引用**：LuaBenghuai  
**状态**：1217验证通过
```lua
	LuaBenghuai = sgs.CreateTriggerSkill{
		name = "LuaBenghuai",
		frequency = sgs.Skill_Compulsory,
		events = {sgs.EventPhaseStart},
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			local phase = player:getPhase()
			if phase == sgs.Player_Finish then
				local list = room:getOtherPlayers(player)
				local cantrigger = false
				for _,p in sgs.qlist(list) do
					if p:getHp() < player:getHp() then
						cantrigger = true
						break
					end
				end
				if cantrigger then
					local result = room:askForChoice(player, self:objectName(), "hp+maxhp")
					if result == "hp" then
						room:loseHp(player)
					else
						room:loseMaxHp(player)
					end
				end
				return false
			end
		end
	}
```
[返回索引](#技能索引) 

##笔伐
**相关武将**：SP·陈琳  
**描述**：结束阶段开始时，你可以将一张手牌移出游戏并选择一名其他角色，该角色的回合开始时，观看该牌，然后选择一项：交给你一张与该牌类型相同的牌并获得该牌，或将该牌置入弃牌堆并失去1点体力。  
**引用**：LuaBifa  
**状态**：1217验证通过
```lua
	LuaBifaCard = sgs.CreateSkillCard{
		name = "LuaBifa",
		target_fixed = false,
		will_throw = false,
		filter = function(self, targets, to_select)
			if #targets == 0 then
				local bifalist = to_select:getPile("bifa")
				if bifalist:isEmpty() then
					return to_select:objectName() ~= sgs.Self:objectName()
				end
			end
			return false
		end,
		on_use = function(self, room, source, targets)
			local target = targets[1]
			local keystr = string.format("BifaSource%d", self:getEffectiveId())
			local tag = sgs.QVariant()
			tag:setValue(source)
			room:setTag(keystr, tag)
			local cards = self:getSubcards()
			for _,id in sgs.qlist(cards) do
				target:addToPile("bifa", id, false)
			end
		end
	}
	LuaBifaVS = sgs.CreateViewAsSkill{
		name = "LuaBifa",
		n = 1,
		view_filter = function(self, selected, to_select)
			return not to_select:isEquipped()
		end,
		view_as = function(self, cards)
			if #cards == 1 then
				local card = LuaBifaCard:clone()
				card:addSubcard(cards[1])
				return card
			end
		end,
		enabled_at_play = function(self, player)
			return false
		end,
		enabled_at_response = function(self, player, pattern)
			return pattern == "@@LuaBifa"
		end
	}
	LuaBifa = sgs.CreateTriggerSkill{
		name = "LuaBifa",
		frequency = sgs.Skill_NotFrequent,
		events = {sgs.EventPhaseStart},
		view_as_skill = LuaBifaVS,
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			if player:isAlive() and player:hasSkill(self:objectName()) then
				if player:getPhase() == sgs.Player_Finish then
					if not player:isKongcheng() then
						room:askForUseCard(player, "@@LuaBifa", "@bifa-remove")
						return false
					end
				end
			end
			if player:getPhase() == sgs.Player_RoundStart then
				local bifa_list = player:getPile("bifa")
				if bifa_list:length() > 0 then
					while not bifa_list:isEmpty() do
						local card_id = bifa_list:first()
						local keystr = string.format("BifaSource%d", card_id)
						local tag = room:getTag(keystr)
						local chenlin = tag:toPlayer()
						local ids = sgs.IntList()
						ids:append(card_id)
						room:fillAG(ids, player)
						local cd = sgs.Sanguosha:getCard(card_id)
						local pattern
						if cd:isKindOf("BasicCard") then
							pattern = "BasicCard"
						elseif cd:isKindOf("TrickCard") then
							pattern = "TrickCard"
						elseif cd:isKindOf("EquipCard") then
							pattern = "EquipCard"
						end
						local data_for_ai = sgs.QVariant(pattern)
						pattern = string.format("%s|.|.|hand", pattern)
						local to_give = nil
						if not player:isKongcheng() and chenlin and chenlin:isAlive() then
							to_give = room:askForCard(player, pattern, "@bifa-give", data_for_ai, sgs.NonTrigger, chenlin)
						end
						if to_give then						
							local reasonG = sgs.CardMoveReason(sgs.CardMoveReason_S_REASON_GIVE, player:objectName(), chenlin:objectName(),self:objectName(), "")
							room:moveCardTo(to_give, player, chenlin, sgs.Player_PlaceHand, reasonG, false)
							local reason = sgs.CardMoveReason(sgs.CardMoveReason_S_REASON_EXCHANGE_FROM_PILE, player:objectName(), self:objectName(), "")
							room:moveCardTo(cd, nil, player, sgs.Player_PlaceHand, reason, false)						
						else
							local reason = sgs.CardMoveReason(sgs.CardMoveReason_S_REASON_REMOVE_FROM_PILE, "", self:objectName(), "")
							room:throwCard(cd, reason, nil)
							room:loseHp(player)
						end
						bifa_list:removeOne(card_id)
						player:invoke("clearAG")
						room:removeTag(keystr)
					end
				end
			end
			return false
		end,
		can_trigger = function(self, target)
			return target
		end
	}
```
[返回索引](#技能索引) 

##闭月
**相关武将**：标准·貂蝉、SP貂蝉、☆SP貂蝉、1v1·貂蝉1v1、怀旧-标准·貂蝉-旧、SP·台版貂蝉  
**描述**：结束阶段开始时，你可以摸一张牌。  
**引用**：LuaBiyue  
**状态**：1217验证通过
```lua
	LuaBiyue = sgs.CreateTriggerSkill{
		name = "LuaBiyue",
		frequency = sgs.Skill_Frequent,
		events = {sgs.EventPhaseStart},
		on_trigger = function(self, event, player, data)
			if player:getPhase() == sgs.Player_Finish then
				local room = player:getRoom()
				if room:askForSkillInvoke(player, self:objectName()) then
					player:drawCards(1)
				end
			end
		end
	}
```
[返回索引](#技能索引) 

##补益
**相关武将**：一将成名·吴国太  
**描述**：当一名角色进入濒死状态时，你可以展示该角色的一张手牌，若此牌不为基本牌，该角色弃置之，然后回复1点体力。  
**引用**：LuaBuyi  
**状态**：1217验证通过
```lua
	LuaBuyi = sgs.CreateTriggerSkill{
		name = "LuaBuyi",
		events = {sgs.Dying},
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			local dying = data:toDying()
			local _player = dying.who
			if _player:isKongcheng() then return false end
			if _player:getHp() < 1 then
				if room:askForSkillInvoke(player, self:objectName(), data) then
					local card
					if player:objectName() == _player:objectName() then
						card = room:askForCardShow(_player, player, "LuaBuyi")
					else
						local id = room:askForCardChosen(player, _player, "h", self:objectName())
						card = sgs.Sanguosha:getCard(id)
					end
					room:showCard(_player, card:getEffectiveId())
					if card:getTypeId() ~= sgs.Card_TypeBasic then
						if not _player:isJilei(card) then
							room:throwCard(card, _player)
						end
						local recover = sgs.RecoverStruct()
						recover.who = player
						room:recover(_player, recover)
					end
				end
			end
			return false
		end,
	}
```
[返回索引](#技能索引) 

##不屈
**相关武将**：风·周泰  
**描述**：**锁定技，**每当你处于濒死状态时，你将牌堆顶的一张牌置于武将牌上：若无同点数的“不屈牌”，你回复至1点体力；否则你将该牌置入弃牌堆。若你有“不屈牌”，你的手牌上限等于“不屈牌”的数量。   
**引用**：LuaBuqu、LuaBuquRemove  
**状态**：1217验证通过
```lua
	LuaBuqu = sgs.CreateTriggerSkill{
		name = "LuaBuqu" ,
		events = {sgs.AskForPeaches} ,
		frequency = sgs.Skill_Compulsory ,
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
	        local dying_data = data:toDying()
			if dying_data.who:objectName() ~= player:objectName() then
	        	return false
			end
	    	if player:getHp() > 0 then return false end
	    	local log = sgs.LogMessage()
	    	log.type = "#TriggerSkill"
	    	log.from = player
	    	log.arg = self:objectName()
	    	room:sendLog(log)
	    	local id = room:drawCard()
	    	local num = sgs.Sanguosha:getCard(id):getNumber()
	    	local duplicate = false
	    	for _, card_id in sgs.qlist(player:getPile("buqu")) do
	    		if sgs.Sanguosha:getCard(card_id):getNumber() == num then
	    			duplicate = true
	            	break
	    		end
	    	end
	    	player:addToPile("buqu", id)
	    	if duplicate then
	    		local reason = sgs.CardMoveReason(sgs.CardMoveReason_S_REASON_REMOVE_FROM_PILE, "", self:objectName(), "")
	    		room:throwCard(sgs.Sanguosha:getCard(id), reason, nil)
	    	else
	    		local recover = sgs.RecoverStruct()
	    		recover.who = player
	    		recover.recover = 1 - player:getHp()
	    		room:recover(player, recover)
	    	end
	    	return false
	    end
	}
	LuaBuquMaxCards = sgs.CreateMaxCardsSkill{
		name = "#LuaBuqu" ,
		fixed_func = function(self, target)
	    	local len = target:getPile("buqu"):length()
	    	if len > 0 then
	        	return len
	    	else
	    		return -1
			end
		end
	}
```
[返回索引](#技能索引) 

##不屈-旧风
**相关武将**：怀旧·周泰  
**描述**：每当你扣减1点体力后，若你的体力值为0，你可以将牌堆顶的一张牌置于武将牌上，称为“创”，若所有“创”的点数均不同，你不会进入濒死状态。  
**引用**：LuaBuqu、LuaBuquRemove  
**状态**：1217验证通过
```lua
	function Remove(SP)
		local room = SP:getRoom()
		local card_ids = SP:getPile("nosbuqu")
		local re = sgs.CardMoveReason(sgs.CardMoveReason_S_REASON_REMOVE_FROM_PILE, "", "LuaNosBuqu", "")
		local lack = 1 - SP:getHp()
		if lack <= 0 then
			for _,id in sgs.qlist(card_ids) do
				local card = sgs.Sanguosha:getCard(id)
				room:throwCard(card, re, nil)
			end
		else
			local to_remove = card_ids:length() - lack
			for var = 1, to_remove do
				if not card_ids:isEmpty() then
					room:fillAG(card_ids)
					local card_id = room:askForAG(SP, card_ids, false, "LuaNosBuqu")
					if card_id ~= -1 then
						card_ids:removeOne(card_id)
						room:throwCard(sgs.Sanguosha:getCard(card_id), re, nil)
					end
					room:broadcastInvoke("clearAG")
				end
			end
		end
	end
	LuaNosBuqu = sgs.CreateTriggerSkill{
		name = "LuaNosBuqu",
		events = {sgs.PostHpReduced, sgs.AskForPeachesDone},
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			if event == sgs.PostHpReduced then
				if player:getHp() < 1 then
					if room:askForSkillInvoke(player, self:objectName(),data) then
						room:setTag(self:objectName(), sgs.QVariant(player:objectName()))
						local buqu = player:getPile("nosbuqu")
						local lack = 1 - player:getHp()
						local n = lack - buqu:length()
						if n > 0 then
							local CAS = room:getNCards(n, false)
							for _,id in sgs.qlist(CAS) do
								player:addToPile("nosbuqu", id)
							end
						end
						local buqun = player:getPile("nosbuqu")
						local duplicate_numbers = sgs.IntList()
						local nub = {}
						for _,id in sgs.qlist(buqun) do
							local card = sgs.Sanguosha:getCard(id)
							local Nm = card:getNumber()
							if table.contains(nub, Nm) then
								duplicate_numbers:append(Nm)
							else
								table.insert(nub, Nm)
							end
						end
						if duplicate_numbers:isEmpty() then
							room:setTag(self:objectName(), sgs.QVariant())
							return true
						end
					end
				end
			elseif event == sgs.AskForPeachesDone then
				local buqun = player:getPile("nosbuqu")
				if player:getHp() > 0 then
					return
				end
				if room:getTag(self:objectName()):toString() ~= player:objectName() then
					return
				end
				local duplicate_numbers = sgs.IntList()
				local nub = {}
				for _,id in sgs.qlist(buqun) do
					local card = sgs.Sanguosha:getCard(id)
					local Nm = card:getNumber()
					if table.contains(nub, Nm) and not duplicate_numbers:contains(Nm) then
						duplicate_numbers:append(Nm)
					else
						table.insert(nub, Nm)
					end
				end
				if duplicate_numbers:isEmpty() then
					room:setPlayerFlag(player, "-dying")
					return true
				end
			end
		end
	}
	LuaNosBuquRemove = sgs.CreateTriggerSkill{
		name = "#LuaNosBuquRemove",
		events = {sgs.HpRecover, sgs.EventLoseSkill},
		on_trigger = function(self, event, player, data)
			local room = player:getRoom()
			if event == sgs.HpRecover then
				if player:hasSkill("LuaNosBuqu") then
					if player:getPile("nosbuqu"):length() > 0 then
						Remove(player)
					end
				end
			elseif event == sgs.EventLoseSkill then
				if data:toString() == "LuaNosBuqu" then
					player:removePileByName("nosbuqu")
					if player:getHp() <= 0 then
						room:enterDying(player, nil)
					end
				end
			end
		end,
		can_trigger = function(self, target)
			return target
		end
	}
```
[返回索引](#技能索引) 