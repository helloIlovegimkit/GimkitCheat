<script lang="ts">
    import Group from "../../hud/Group.svelte";
    import PurchaseAnywhere from "../shared/PurchaseAnywhere.svelte";
	let lastPos = {
		x: null,
		y: null
	}

	gc.socket.onOutgoingMsg((type, msg) => {
		if(type != "MOVED") return
		lastPos = msg
	})

	let autoAttacking = false
	let attackBtn = gc.hud.addToggleBtn("Stop auto attacking", "Auto Attack", (state) => {
		autoAttacking = state
	}, false)

	keybinds.addKeybindSetter("Auto Attack", () => {
		attackBtn.trigger()
	})
	
	setInterval(() => {
		if(!autoAttacking) return
		let characters = JSON.parse(JSON.stringify(gc.data.serializer.getState().characters))

		// calculate the closest sentry to the last position we were at
		let target
		let shortedDistance = Infinity
		for(let id in characters) {
			let character = characters[id]
			if(character.type != "sentry" || character.isRespawning) continue
			let distance = Math.sqrt(Math.pow(character.x - lastPos.x, 2) + Math.pow(character.y - lastPos.y, 2))
			if(distance < shortedDistance) {
				target = character
				shortedDistance = distance
			}
		}

		if(!target) return
		gc.socket.sendObj("FIRE", {
			angle: 0,
			x: target.x,
			y: target.y
		})
	}, 100)
</script>

<Group name="One Way Out">
    <PurchaseAnywhere displayText="Med Pack" reusable={true}
        selector={{ options: { grantedItemId: "medpack" }}} />
    <PurchaseAnywhere displayText="Shield Can" reusable={true}
        selector={{ options: { grantedItemId: "shield-can" }}} />
</Group>

            
