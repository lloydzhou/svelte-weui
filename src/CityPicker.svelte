<svelte:options tag="weui-city-picker" />

<script>
  import { onMount } from 'svelte'
  export let show = false
  export let cancelBtn = '取消'
  export let okBtn = '确定'
  export let province = ''
  export let city = ''
  export let area = ''

  let root = arguments[0]
  let level = []
  let groups = [{items: []}, {items: []}, {items: []}]

  onMount(() => {
    fetch('https://unpkg.com/province-city-china/dist/level.json').then(res => res.json()).then(data => {
      level = data
    })
  })

  $: {
    // groupIndex: 0
    // console.log('province', province)
    groups[0].items = level.map(i => ({label: i.name, code: i.code}))
    let index0 = index(groups[0].items, province)
    if (groups[0].items.length > 0 && index0 == -1) {
      index0 = 0
    }
    province = index0 >= 0 ? groups[0].items[index0].label : ''
    groups[0].defaultIndex = index0
    // console.log('province', index0, province)
    // groupIndex: 1
    if (index0 >= 0) {
      groups[1].items = (level[index0].children || []).map(i => ({label: i.name, code: i.code}))
      // console.log('city', city)
      let index1 = index(groups[1].items, city)
      if (groups[1].items.length > 0 && index1 == -1) {
        index1 = 0
      }
      city = index1 >= 0 ? groups[1].items[index1].label : ''
      groups[1].defaultIndex = index1
      // console.log('city', index1, city)
      // groupIndex: 2
      if (index1 >= 0) {
        groups[2].items = ((level[index0].children || [])[index1].children || []).map(i => ({label: i.name, code: i.code}))
        let index2 = index(groups[2].items, area)
        if (groups[2].items.length > 0 && index2 == -1) {
          index2 = 0
        }
        area = index2 >= 0 ? groups[2].items[index2].label : ''
        groups[2].defaultIndex = index2
        // console.log('area', index2, area)
      }
    }
  }

  function index(items, name) {
    for (const index in items) {
      if (items[index].label == name) {
        return index
      }
    }
    return -1
  }

  function groupchange(e) {
    // console.debug('groupchange', e.detail, root)
    const { groupIndex, item, selected } = e.detail
    if (selected >= 0 && groupIndex == 0 && province != item.label) {
      province = item.label
    }
    if (selected >= 0 && groupIndex == 1 && city != item.label) {
      city = item.label
    }
    if (selected >= 0 && groupIndex == 2 && area != item.label) {
      area = item.label
    }
    root.dispatchEvent(custom_event('groupchange', e.detail))
  }

  function change(e) {
    root.dispatchEvent(custom_event('change', {province, city, area}))
  }

  function custom_event(type, detail) {
    const e = document.createEvent('CustomEvent');
    e.initCustomEvent(type, false, false, detail);
    return e;
  }


</script>

<weui-picker on:groupchange={groupchange} on:change={change} groups={groups} show={show} cancelBtn={cancelBtn} okBtn={okBtn}>
  <slot />
</weui-picker>

