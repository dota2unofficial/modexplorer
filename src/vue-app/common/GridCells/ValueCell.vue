<template>
	<div>
		<div v-if="isAbilitySpecial">
			<div v-for="(item, index) in data" :key="index" class="table">
				<span>{{ getTitle(item) }}</span>
				<span v-if="!getHideValueType">{{ getKey(index) }}</span>
				<span>{{ getValue(index) }}</span>
			</div>
		</div>
		<div v-else-if="isAbility" class="d-flex align-center">
			<div class="d-flex align-center" v-if="hasIcon">
				<img
					width="24px"
					height="24px"
					:src="getAbilityIcon"
					class="mr-2"
					alt="skill"
				/>
			</div>
			<div>{{ getAbility ? getFormattedValue : this.params.value }}</div>
		</div>
		<span v-else>{{ getFormattedValue }}</span>
	</div>
</template>

<script>
import Vue from "vue";
import { mapGetters, mapMutations } from "vuex";
import { flatten } from "../../utils/file";

import fs from "fs";

export default Vue.extend({
	name: "ValueCell",
	data: () => ({
		data: [],
		models: []
	}),
	created() {
		Object.keys(this.params.value).forEach(key => {
			let newValue = {};
			Object.keys(this.params.value[key]).forEach(index => {
				newValue[index] = this.params.value[key][index];
			});
			this.data.push(newValue);
			this.models.push(newValue);
		});
	},
	computed: {
		...mapGetters([
			"getLocalizationData",
			"getCustomLocalization",
			"getHideValueType",
			"getAbility",
			"getHeros",
			"getAbilities"
		]),
		isAbilitySpecial() {
			if (
				this.params.data.key !== "AbilitySpecial" &&
				this.params.data.key !== "Creature"
			)
				return false;
			return true;
		},
		isAbility() {
			return /Ability\d/.test(this.params.data.key);
		},
		getKeys() {
			return Object.keys(flatten(this.params.value));
		},
		getAbilityIcon() {
			const entity = this.params.value;
			const defaultPath = `file:\\${process.cwd()}\\${
				process.env.NODE_ENV === "development" ? "" : "resources\\"
			}assets`;

			const currentEntity = this.getAbilities[entity];
			if (currentEntity && currentEntity["AbilityTextureName"]) {
				const defaultD2Res = `${this.getD2Path}\\dota_addons\\${this.getPath}\\resource\\flash3\\images\\spellicons\\${currentEntity["AbilityTextureName"]}`;
				const localizationData = this.getLocalizationData[
					`DOTA_Tooltip_ability_${currentEntity["AbilityTextureName"]}`
				];
				if (fs.existsSync(`${defaultD2Res}_lua.png`))
					return `${defaultD2Res}_lua.png`;
				else if (fs.existsSync(`${defaultD2Res}.png`))
					return `${defaultD2Res}.png`;
				else if (localizationData) {
					const localizedIcon = `${defaultPath}\\spells\\${localizationData
						.split(" ")
						.join("_")}_icon.png`;

					let retUrl;
					try {
						fs.accessSync(localizedIcon);
						return localizedIcon;
					} catch {
						retUrl = `${defaultPath}\\spells\\${currentEntity["AbilityTextureName"]}.png`;
					}
					return retUrl;
				} else
					return `${defaultPath}\\spells\\${currentEntity["AbilityTextureName"]}.png`;
			}
			return `${defaultPath}\\spells\\${entity}.png`;
		},
		hasIcon() {
			const data = this.getLocalizationData[
				`DOTA_Tooltip_ability_${this.params.value}`
			]
				? this.getLocalizationData[
						`DOTA_Tooltip_ability_${this.params.value}`
				  ]
				: this.getCustomLocalization[
						`DOTA_Tooltip_ability_${this.params.value}`
				  ];
			return data === undefined
				? false
				: data.indexOf("{s:value}") === -1;
		},
		getFormattedValue() {
			if (this.params.data.key === "AbilityTextureName")
				return this.params.value;
			if (
				this.getCustomLocalization[
					`DOTA_Tooltip_ability_${this.params.value}`
				]
			)
				return this.getCustomLocalization[
					`DOTA_Tooltip_ability_${this.params.value}`
				].replace("{s:value}", this.params.value.split("_").pop());
			return this.getLocalizationData[
				`DOTA_Tooltip_ability_${this.params.value}`
			]
				? this.getLocalizationData[
						`DOTA_Tooltip_ability_${this.params.value}`
				  ].replace("{s:value}", this.params.value.split("_").pop())
				: this.params.value;
		}
	},
	methods: {
		getValue(index) {
			return this.models[index][Object.keys(this.models[index])[1]];
		},
		getKey(index) {
			return this.models[index][Object.keys(this.models[index])[0]];
		},
		getFlattenArray(item) {
			return Object.keys(item);
		},
		getTitle(item) {
			const str = this.getFlattenArray(item)[1];
			if (str) return str.toUpperCase();
			return "";
		}
	}
});
</script>

<style lang="scss" scoped>
.default {
	color: #aaa !important;
}
.table {
	display: flex;
	position: relative;

	&.border-top {
		&:before {
			content: "";
			height: 1px;
			position: absolute;
			left: -17px;
			right: -17px;
			top: -1px;
			background-color: #000;
		}
	}

	span {
		flex: 50%;
		position: relative;
		overflow: hidden;

		&:not(:last-child) {
			&:after {
				content: "";
				position: absolute;
				right: 8px;
				top: 8px;
				width: 1px;
				bottom: 8px;
				background-color: #ccc;
			}
		}

		&:before {
			content: "";
			position: absolute;
			left: 0;
			width: 100%;
			bottom: 0;
			height: 1px;
			background-color: #ccc;
		}
	}
}
</style>
