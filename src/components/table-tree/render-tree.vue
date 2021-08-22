<!--
 * @Author: pimzh
 * @Date: 2021-08-19 19:17:54
 * @LastEditTime: 2021-08-22 14:01:09
 * @LastEditors: pimzh
 * @Description:
-->
<script>
export default {
  name: 'RenderTree',
  functional: true,
  render(_, context) {
    const { data, onIconClick } = context.props

    const renderTemp = () => {
      const arr = []
      let len = data.__level
      while (len--) {
        arr.push(_('span', { class: 'placeholder' }))
      }
      // render icon
      arr.push(
        _('span', {
          class: 'icon-wrapper' + (data.__isLast ? ' icon-wrapper-last' : '')
        }, [
          data.children?.length && _('i', {
            class: 'icon-cls ' + (data.expand ? 'el-icon-remove' : 'el-icon-circle-plus'),
            on: {
              click: (e) => {
                e.stopPropagation()                
                onIconClick(data)
              }
            }
          }) || ''
        ])
      )
      return arr
    }

    return renderTemp()
  }
}
</script>

<style lang="scss" scoped>
*::after, *::before {
  box-sizing: border-box;
}
.icon-wrapper {
  position: relative;
  flex: 1;
  height: 100%;
  display: inline-block;
  .icon-cls {
    display: inline-block;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    left: 3px;
    z-index: 2;
    background-color: #ffffff;
  }
  &::before {
    content: '';
    position: absolute;
    display: inline-block;
    top: 0;
    left: 9px;
    border-left: 1px dashed black;
    width: 2px;
    height: 100%;
    z-index: 1;
  }
  &::after {
    content: '';
    position: absolute;
    display: inline-block;
    width: 100%;
    height: 2px;
    left: 10px;
    top: 50%;
    transform: translateY(-50%);
    border-top: 1px dashed black;
    z-index: 1;
  }
}
.icon-wrapper-last {
  &::before {
    top: -25px;
  }
}
.placeholder {
  display: inline-block;
  width: 20px;
  height: 100%;
  position: relative;
  &::after {
    content: "";
    display: inline-block;
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 2px;
    height: 100%;
    border-left: 1px dashed black;
  }
}
</style>
