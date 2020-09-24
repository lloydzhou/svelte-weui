<svelte:options tag="weui-picker" />

<script>
  import { createEventDispatcher } from 'svelte'
  export let groups = []
  export let show = false
  export let cancelBtn = '取消'
  export let okBtn = '确定'

  const dispatch = createEventDispatcher()
  let root = arguments[0]
  let selected = []
  let closing = false

  $: _groups = (typeof groups == "string" ? JSON.parse(groups) : groups)

  // picker-group
  function initgroup(node, group) {
    const defaultProps = {
      height: 238,
      itemHeight: 25 + 9, //content + padding
      indicatorTop: 102,
      indicatorHeight: 34,
      aniamtion: true,
      groupIndex: -1,
      defaultIndex: -1,
    }
    const content = node.querySelector('.weui-picker__content')
    let props = {...defaultProps, ...group}
    const state = {
      touching: false,
      ogY: 0,
      ogTranslate: 0,
      touchId: undefined,
      translate: 0,
      totalHeight: 0,
      selected: 0,
      animating: group.animation
    }

    function adjustPosition(){
      const { items, itemHeight, indicatorTop, defaultIndex, groupIndex } = props
      const totalHeight = items.length * itemHeight;
      let translate = totalHeight <= indicatorTop ? indicatorTop : 0;
      if (defaultIndex > -1) {
        if (translate === 0){
          let upperCount = Math.floor(indicatorTop / itemHeight);
          if (defaultIndex > upperCount){
            //over
            let overCount = defaultIndex - upperCount;
            translate -= overCount * itemHeight;
          } else if (defaultIndex === upperCount){
            translate = 0;
          } else {
            //less
            translate += (Math.abs(upperCount - defaultIndex) * itemHeight);
          }
        } else {
          //total item less than indicator height
          translate -= itemHeight * defaultIndex;
        }
      }
      content.style.transform = `translate(0, ${translate}px)`
      content.style.transition = state.animating ? 'transform .3s' : 'none'
      Object.assign(state, {selected: defaultIndex, ogTranslate: translate, totalHeight, translate})
      updateSelected(defaultIndex > -1 ? false : true)
    }
    function updateSelected(propagate=true) {
      const { items=[], itemHeight, indicatorTop, indicatorHeight, groupIndex } = props
      const { translate=0 } = state
      items.map((item, i) => {
        if (!item.disabled && (translate + (itemHeight * i)) >= indicatorTop && (translate + (itemHeight * i) + itemHeight ) <= indicatorTop + indicatorHeight){
          state.selected = i;
        }
      })
      selected[groupIndex] = state.selected
      if (propagate) {
        dispatch('groupchange', {
          item: items[state.selected],
          selected: state.selected,
          groupIndex
        })
      }
    }
    function handleTouchStart(e){
      if (state.touching || props.items.length <= 1) return;
      Object.assign(state, {
        touching: true,
        ogTranslate: state.translate,
        touchId: e.targetTouches[0].identifier,
        ogY: state.translate === 0 ? e.targetTouches[0].pageY : e.targetTouches[0].pageY - state.translate,
        animating: false
      })
    }
    function handleTouchMove(e){
      if (!state.touching || props.items.length <= 1) return;
      if (e.targetTouches[0].identifier !== state.touchId) return;
      //prevent move background
      e.preventDefault();
      const pageY = e.targetTouches[0].pageY;
      const diffY = pageY - state.ogY;
      state.translate = diffY
      content.style.transform = `translate(0, ${state.translate}px)`
    }

    function handleTouchEnd(e){
      if (!state.touching || props.items.length <= 1) return;
      const { indicatorTop, indicatorHeight, itemHeight } = props;
      let translate = state.translate;

      if (Math.abs(translate - state.ogTranslate) < (itemHeight * .51) ){
        translate = state.ogTranslate;
      } else if (translate > indicatorTop) {
        //top boundry
        translate = indicatorTop;
      } else if (translate + state.totalHeight < indicatorTop + indicatorHeight) {
        //bottom
        translate = indicatorTop + indicatorHeight - state.totalHeight;
      } else {
        //pass single item range but not exceed boundry
        let step = 0, adjust = 0;
        let diff = (translate - state.ogTranslate) / itemHeight;
        if (Math.abs(diff) < 1){
          step = diff > 0 ? 1 : -1;
        } else {
          adjust = Math.abs((diff % 1) * 100) > 50 ? 1 : 0;
          step = diff > 0 ? Math.floor(diff) + adjust : Math.ceil(diff) - adjust;
        }
        translate = state.ogTranslate + ( step * itemHeight );
      }
      Object.assign(state, {
          touching: false,
          ogY: 0,
          touchId: undefined,
          ogTranslate: 0,
          animating: true,
          translate
      })
      content.style.transform = `translate(0, ${state.translate}px)`
      updateSelected(true)
    }

    adjustPosition()
    //console.log('use touch', node, props, state, content)
    const touchStartHandler = handleTouchStart.bind(node)
    const touchMoveHandler = handleTouchMove.bind(node)
    const touchEndHandler = handleTouchEnd.bind(node)
    node.addEventListener('touchstart', touchStartHandler, false)
    node.addEventListener('touchmove', touchMoveHandler, false)
    node.addEventListener('touchend', touchEndHandler, false)
    return {
      update(group) {
        //console.log('use touch update', group)
        Object.assign(props, group)
        adjustPosition(props, state, content)
      },
      destroy() {
        node.removeEventListener('touchstart', touchStartHandler, false)
        node.removeEventListener('touchmove', touchMoveHandler, false)
        node.removeEventListener('touchend', touchEndHandler, false)
      }
    }
  }
  // picker-group
  // picker function
  function handleClose(cb) {
    closing = true
    setTimeout(() => {
      closing = false
      dispatch('close', {})
      cb && cb()
    })
  }
  function handleChange(e) {
    handleClose(() => {
      dispatch('change', {selected})
    })
  }
  // picker function

  root.$on('groupchange', function(e) {
    console.debug('onGroupChange', e.detail)
    const { item, selected: index, groupIndex } = e.detail
    selected[groupIndex] = index
    root.dispatchEvent(e)
  })

  root.$on('cancel', function(e) {
    console.debug('onCancel', e.detail)
    root.dispatchEvent(e)
  })

  root.$on('change', function(e) {
    console.debug('onChange', e.detail)
    root.dispatchEvent(e)
  })

  root.$on('close', function(e) {
    console.debug('onClose', e.detail)
    show = false
    root.dispatchEvent(e)
  })

  root.$on('open', function(e) {
    console.log('onOpen', e.detail)
    show = true
    root.dispatchEvent(e)
  })

</script>

<div on:click={e => dispatch('open', {e})}>
  {#if show}
  <div class="weui_picker">
    <weui-mask on:click|stopPropagation={e => handleClose()} />
    <div class="weui-picker" class:weui-animate-slide-up={show && !closing} class:weui-animate-slide-down={closing}>
      <div class="weui-picker__hd">
        <div class="weui-picker__action" on:click|stopPropagation={e => handleClose()}>{cancelBtn}</div>
        <div class="weui-picker__action" on:click|stopPropagation={e => handleChange()}>{okBtn}</div>
      </div>
      <div class="weui-picker__bd">
        {#each _groups as group, groupIndex}
          <div class="weui-picker__group" use:initgroup={{...group, groupIndex}}>
            <div class="weui-picker__mask"></div>
            <div class="weui-picker__indicator"></div>
            <div class="weui-picker__content">
              {#each (group.items||[]) as item}
                <div class="weui-picker__item" class:weui-picker__item_disabled={item.disabled}>
                  {item.label}
                </div>
              {/each}
            </div>
          </div>
        {/each}
      </div>
    </div>
  </div>
  {/if}
  
  <slot />
</div>

<style>
.weui-picker {
  font-size: 16px;
  position: fixed;
  width: 100%;
  left: 0;
  bottom: 0;
  z-index: 5000;
  -webkit-backface-visibility: hidden;
          backface-visibility: hidden;
  -webkit-transform: translate(0, 100%);
          transform: translate(0, 100%);
  -webkit-transition: -webkit-transform .3s;
  transition: -webkit-transform .3s;
  transition: transform .3s;
  transition: transform .3s, -webkit-transform .3s;
}
.weui-picker__hd {
  display: -webkit-box;
  display: -webkit-flex;
  display: flex;
  padding: 10px 15px;
  background-color: #fbf9fe;
  position: relative;
  text-align: center;
}
.weui-picker__hd:after {
  content: " ";
  position: absolute;
  left: 0;
  bottom: 0;
  right: 0;
  height: 1px;
  border-bottom: 1px solid #E5E5E5;
  color: #E5E5E5;
  -webkit-transform-origin: 0 100%;
          transform-origin: 0 100%;
  -webkit-transform: scaleY(0.5);
          transform: scaleY(0.5);
}
.weui-picker__action {
  display: block;
  -webkit-box-flex: 1;
  -webkit-flex: 1;
          flex: 1;
  //color: #586C94;
  color: #333333;
  height: 25px;
  line-height: 25px;
}
.weui-picker__action:first-child {
  text-align: left;
}
.weui-picker__action:last-child {
  text-align: right;
}
.weui-picker__bd {
  display: -webkit-box;
  display: -webkit-flex;
  display: flex;
  position: relative;
  background-color: #fff;
  height: 238px;
  overflow: hidden;
}
.weui-picker__group {
  -webkit-box-flex: 1;
  -webkit-flex: 1;
          flex: 1;
  position: relative;
  height: 100%;
}
.weui-picker__mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  margin: 0 auto;
  z-index: 3;
  background: -webkit-linear-gradient(top, rgba(255, 255, 255, 0.95), rgba(255, 255, 255, 0.6)), -webkit-linear-gradient(bottom, rgba(255, 255, 255, 0.95), rgba(255, 255, 255, 0.6));
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.95), rgba(255, 255, 255, 0.6)), linear-gradient(0deg, rgba(255, 255, 255, 0.95), rgba(255, 255, 255, 0.6));
  background-position: top, bottom;
  background-size: 100% 102px;
  background-repeat: no-repeat;
  -webkit-transform: translateZ(0);
          transform: translateZ(0);
}
.weui-picker__indicator {
  width: 100%;
  height: 34px;
  position: absolute;
  left: 0;
  top: 102px;
  z-index: 3;
}
.weui-picker__indicator:before {
  content: " ";
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  height: 1px;
  border-top: 1px solid #E5E5E5;
  color: #E5E5E5;
  -webkit-transform-origin: 0 0;
          transform-origin: 0 0;
  -webkit-transform: scaleY(0.5);
          transform: scaleY(0.5);
}
.weui-picker__indicator:after {
  content: " ";
  position: absolute;
  left: 0;
  bottom: 0;
  right: 0;
  height: 1px;
  border-bottom: 1px solid #E5E5E5;
  color: #E5E5E5;
  -webkit-transform-origin: 0 100%;
          transform-origin: 0 100%;
  -webkit-transform: scaleY(0.5);
          transform: scaleY(0.5);
}
.weui-picker__content {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
.weui-picker__item {
  padding: 5px 0 4px;
  text-align: center;
  color: #000;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
  height: 25px;
  line-height: 25px;
}
.weui-picker__item_disabled {
  color: #999999;
}
@-webkit-keyframes slideUp {
  from {
    -webkit-transform: translate3d(0, 100%, 0);
            transform: translate3d(0, 100%, 0);
  }
  to {
    -webkit-transform: translate3d(0, 0, 0);
            transform: translate3d(0, 0, 0);
  }
}
@keyframes slideUp {
  from {
    -webkit-transform: translate3d(0, 100%, 0);
            transform: translate3d(0, 100%, 0);
  }
  to {
    -webkit-transform: translate3d(0, 0, 0);
            transform: translate3d(0, 0, 0);
  }
}
.weui-animate-slide-up {
  -webkit-animation: slideUp ease .3s forwards;
          animation: slideUp ease .3s forwards;
}
</style>
