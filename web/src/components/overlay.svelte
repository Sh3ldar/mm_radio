<script lang="ts">
    import { SendEvent } from '@utils/eventsHandlers';
    import { ReceiveEvent } from '@utils/eventsHandlers';
    import { RADIODATA, SHOW, SHOWPLAYERLIST, PLAYERLIST, SHOWFORCEPLAYERLIST } from "@stores/stores";
    import { Receive, Send } from '@enums/events';

    let isPlayerListVisible: boolean;

    SHOWPLAYERLIST.subscribe((visible: boolean) => {
		isPlayerListVisible = visible && $RADIODATA.overlay != 'never'
	})

    ReceiveEvent(Receive.updateRadioList, (data: any) => {
		PLAYERLIST.set(data)
	})

    ReceiveEvent(Receive.updateRadioTalking, (data: any) => {
		$PLAYERLIST[data.radioId].isTalking = data.radioTalking
	})

    RADIODATA.subscribe((data: any) => {
        if (data != null && !data.onRadio) 
            PLAYERLIST.set([])
    })
    
    function dragMe(node: any) {
        let moving = false;
        let right = parseInt((node.style.right).replace('px', ''));
        let top = parseInt((node.style.top).replace('px', ''));
        
        node.style.position = 'absolute';
        node.style.top = `${top}px`;
        node.style.right = `${right}px`;
        node.style.userSelect = 'none';
        
        node.addEventListener('mousedown', () => {
            if ($SHOW) 
            moving = true;
        });
        
        window.addEventListener('mousemove', (e) => {
            if (moving) {
                right -= e.movementX;
                top += e.movementY;

                right = Math.min(Math.max(right, 0), window.innerWidth - node.offsetWidth);
                top = Math.min(Math.max(top, 0), window.innerHeight - node.offsetHeight);

                node.style.top = `${top}px`;
                node.style.right = `${right}px`;

                RADIODATA.update((old: any) => {
                    return { ...old, userData: { ...old.userData, playerlist: { ...old.userData.playerlist, coords: { x: right, y: top } } } }
                })
            }
        });
        
        window.addEventListener('mouseup', () => {
            moving = false;
            SendEvent(Send.updatePlayerPosition, { y: top, x: right })
        });

        SHOWFORCEPLAYERLIST.subscribe(() => {
            SendEvent(Send.updatePlayerPosition, { y: top, x: right })
        })
    }
</script>

{#if isPlayerListVisible}
    {#if $RADIODATA.onRadio}
        <div class="max-w-[15vw] absolute z-[1000] text-right select-none" style="top: {$RADIODATA.userData.playerlist.coords.y}px; right: {$RADIODATA.userData.playerlist.coords.x}px;cursor:{$SHOW? 'move':'no-drop'}; transform: scale({($RADIODATA.userData.overlaySizeMultiplier*2)/100})" use:dragMe>
            {#each Object.entries($PLAYERLIST) as [id, player], index (id)}
                {#if player.isTalking || $SHOWFORCEPLAYERLIST || $RADIODATA.overlay == 'always'}
                    <div class="text-[1.7vh] font-bold px-2 text-white" style="color: {$RADIODATA.overlay == 'always' && (player.isTalking? "white":"rgba(169, 169, 169, 0.8)")}">{player.name}</div>
                {/if}
            {/each}
        </div>
    {/if}
{/if}