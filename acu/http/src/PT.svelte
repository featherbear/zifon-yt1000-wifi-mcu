<script lang="ts">
  import { send } from "ionicons/icons";

  import Joystick, {
    DIRECTIONS as JoystickDirections,
  } from "./components/Joystick";

  function stateEvent(state: boolean, direction?: JoystickDirections) {
    if (state) updateBuffer(direction);
    else updateBuffer(STOP_BYTE);
    _send();
  }

  const STOP_BYTE = 0xff;
  let WS: WebSocket;

  let val;
  let lastSentWasStop = false;
  let buffer;
  function updateBuffer(nextVal) {
    if (val === nextVal) return;
    buffer = new Uint8Array([(val = nextVal)]);
  }

  let _send = () => {
    if (val == STOP_BYTE) {
      if (lastSentWasStop) return;
      lastSentWasStop = true;
    } else lastSentWasStop = false;

    WS?.send(buffer);
  };
  function connectToWS() {
    WS = new WebSocket(`ws://${location.hostname}:1337`);
    // WS = new WebSocket(`ws://192.168.0.82:1337`);
    let interval;
    WS.addEventListener("open", () => {
      // show things are connected
      interval = setInterval(_send, 400);
    });

    function reset() {
      clearInterval(interval);
      connectToWS();
    }

    WS.addEventListener("error", reset);
  }
  connectToWS();

  let isDualControl = false;
</script>

<div class="container flex-column">
  {#if isDualControl}
    <div class="container flex-row">
      <div class="joystickContainer">
        <Joystick type="y" emitState={stateEvent}>Y</Joystick>
      </div>

      <div class="joystickContainer">
        <Joystick type="x" emitState={stateEvent}>X</Joystick>
      </div>
    </div>
  {:else}
    <div class="joystickContainer">
      <Joystick type="xy" emitState={stateEvent}>XY</Joystick>
    </div>
  {/if}

  <button
    class="button is-info is-outlined"
    on:click={() => (isDualControl = !isDualControl)}>Toggle Mode</button
  >
</div>

<style lang="scss">
  .container {
    display: flex;
    align-items: center;
    justify-content: center;

    &.flex-column {
      flex-direction: column;
    }

    *.flex-row {
      flex-direction: row;
    }
  }
  .joystickContainer > :global(*) {
    max-width: max(600px, 20vw);
  }
</style>
