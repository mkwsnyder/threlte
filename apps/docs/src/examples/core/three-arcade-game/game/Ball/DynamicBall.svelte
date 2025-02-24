<script lang="ts">
	import {
		CoefficientCombineRule,
		type RigidBody as RapierRigidBody
	} from '@dimforge/rapier3d-compat'
	import { Three, useFrame } from '@threlte/core'
	import { AutoColliders, RigidBody } from '@threlte/rapier'
	import { derived } from 'svelte/store'
	import { Mesh } from 'three'
	import { arenaHeight, playerHeight, playerToBorderDistance } from '../config'
	import { gameState } from '../state'
	import { ballGeometry, ballMaterial } from './common'

	const { state, levelIndex, playerPosition, ballPosition, ballRigidBody } = gameState

	let rigidBody: RapierRigidBody | undefined = undefined
	$: if (rigidBody) ballRigidBody.set(rigidBody)

	const map = (value: number, inMin: number, inMax: number, outMin: number, outMax: number) => {
		return ((value - inMin) * (outMax - outMin)) / (inMax - inMin) + outMin
	}

	const ballSpeed = derived(levelIndex, (levelIndex) => {
		return map(levelIndex, 0, 9, 5, 10)
	})

	let ballIsSpawned = false
	const spawnBall = () => {
		if (!rigidBody) return
		ballIsSpawned = true
		const randomSign = Math.random() > 0.5 ? 1 : -1
		const randomX = (randomSign * Math.random() * $ballSpeed) / 2
		rigidBody.applyImpulse({ x: randomX, y: 0, z: -$ballSpeed }, true)
	}

	const startAtPosZ = arenaHeight / 2 - playerHeight - playerToBorderDistance * 2

	const onSensorEnter = () => {
		if ($state === 'playing') {
			state.set('game-over')
		}
	}

	useFrame(() => {
		if (!ballIsSpawned && rigidBody) {
			spawnBall()
			stop()
		}
		const rbTranslation = rigidBody?.translation()
		ballPosition.set({
			x: rbTranslation?.x ?? 0,
			z: rbTranslation?.z ?? 0
		})
	})
</script>

<RigidBody
	bind:rigidBody
	type={'dynamic'}
	on:sensorenter={onSensorEnter}
	enabledTranslations={[true, false, true]}
	position={{
		z: startAtPosZ,
		x: $playerPosition
	}}
>
	<AutoColliders
		shape="ball"
		mass={1}
		friction={0}
		restitution={1}
		restitutionCombineRule={CoefficientCombineRule.Max}
		frictionCombineRule={CoefficientCombineRule.Min}
	>
		<Three type={Mesh}>
			<Three type={ballGeometry} />
			<Three type={ballMaterial} />
		</Three>
	</AutoColliders>
</RigidBody>
