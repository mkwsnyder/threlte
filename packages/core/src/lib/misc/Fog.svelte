<script lang="ts">
  import { onDestroy } from 'svelte'
  import { Fog as ThreeFog } from 'three'
  import { useThrelte } from '../hooks/useThrelte'
  import { useThrelteRoot } from '../hooks/useThrelteRoot'
  import { convertColorRepresentationToColor } from '../lib/colors'
  import type { FogProperties } from '../types/components'

  export let color: FogProperties['color'] = 0xffffff
  export let near: FogProperties['near'] = undefined
  export let far: FogProperties['far'] = undefined

  const { linear } = useThrelteRoot()
  const { scene, invalidate } = useThrelte()

  export const fog = new ThreeFog(convertColorRepresentationToColor(color, $linear), near, far)

  $: {
    if (color !== undefined) fog.color = convertColorRepresentationToColor(color, $linear)
    if (near !== undefined) fog.near = near
    if (far !== undefined) fog.far = far
    invalidate('Fog: props changed')
  }

  scene.fog = fog
  invalidate('Fog: added')

  onDestroy(() => {
    scene.fog = null
    invalidate('Fog: removed')
  })
</script>
