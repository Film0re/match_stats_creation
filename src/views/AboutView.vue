<template>
  <div class="min-h-screen bg-gray-100 py-8 px-4 sm:px-6 lg:px-8">
    <div class="max-w-7xl mx-auto">
      <h1 class="text-3xl font-bold text-center mb-8">LoL Match Data</h1>

      <form @submit.prevent="generateCSV">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-8 mb-8">
          <div
            v-for="(team, teamIndex) in teams"
            :key="teamIndex"
            class="bg-white shadow-md rounded-lg p-6"
          >
            <h2 class="text-2xl font-semibold mb-4">
              {{ teamIndex === 0 ? 'Blue Team' : 'Red Team' }}
            </h2>
            <div class="mb-4">
              <label :for="`team-${teamIndex}-name`" class="block text-sm font-medium text-gray-700"
                >Team Name</label
              >
              <select
                :id="`team-${teamIndex}-name`"
                v-model="team.id"
                @change="loadPlayers(teamIndex)"
                required
                class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md"
              >
                <option value="">Select a team</option>
                <option
                  v-for="teamOption in teamOptions"
                  :key="teamOption.id"
                  :value="teamOption.id"
                >
                  {{ teamOption.name }}
                </option>
              </select>
            </div>
            <div v-for="(player, playerIndex) in team.players" :key="playerIndex" class="mb-4">
              <h3 class="text-lg font-medium mb-2">{{ getRoleName(playerIndex) }}</h3>
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <label
                    :for="`player-${teamIndex}-${playerIndex}-id`"
                    class="block text-sm font-medium text-gray-700"
                    >Player</label
                  >
                  <select
                    :id="`player-${teamIndex}-${playerIndex}-id`"
                    v-model="player.id"
                    required
                    class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md"
                  >
                    <option value="">Select a player</option>
                    <option
                      v-for="playerOption in player.isSub
                        ? allPlayerOptions
                        : playerOptions[teamIndex]"
                      :key="playerOption.id"
                      :value="playerOption.id"
                    >
                      {{ playerOption.name }}
                    </option>
                  </select>
                  <div class="col-span-2 mt-5">
                    <label class="inline-flex items-center">
                      <input
                        type="checkbox"
                        v-model="player.isSub"
                        @change="handleSubChange(teamIndex, playerIndex)"
                        class="form-checkbox h-5 w-5 text-indigo-600"
                      />
                      <span class="ml-2 text-sm text-gray-700">Sub</span>
                    </label>
                  </div>
                </div>
                <div>
                  <label
                    :for="`player-${teamIndex}-${playerIndex}-champion`"
                    class="block text-sm font-medium text-gray-700"
                    >Champion</label
                  >
                  <select
                    :id="`player-${teamIndex}-${playerIndex}-champion`"
                    v-model="player.championId"
                    required
                    class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md"
                  >
                    <option value="">Select a champion</option>
                    <option
                      v-for="champion in championOptions"
                      :key="champion.id"
                      :value="champion.id"
                    >
                      {{ champion.name }}
                    </option>
                  </select>
                </div>
                <div>
                  <label
                    :for="`player-${teamIndex}-${playerIndex}-kills`"
                    class="block text-sm font-medium text-gray-700"
                    >Kills</label
                  >
                  <input
                    type="number"
                    :id="`player-${teamIndex}-${playerIndex}-kills`"
                    v-model.number="player.kills"
                    required
                    min="0"
                    class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
                  />
                </div>
                <div>
                  <label
                    :for="`player-${teamIndex}-${playerIndex}-deaths`"
                    class="block text-sm font-medium text-gray-700"
                    >Deaths</label
                  >
                  <input
                    type="number"
                    :id="`player-${teamIndex}-${playerIndex}-deaths`"
                    v-model.number="player.deaths"
                    required
                    min="0"
                    class="mt-1 block w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
                  />
                </div>
                <div>
                  <label
                    :for="`player-${teamIndex}-${playerIndex}-assists`"
                    class="block text-sm font-medium text-gray-700"
                    >Assists</label
                  >
                  <input
                    type="number"
                    :id="`player-${teamIndex}-${playerIndex}-assists`"
                    v-model.number="player.assists"
                    required
                    min="0"
                    class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
                  />
                </div>
              </div>
            </div>
            <div class="mt-4">
              <input
                type="radio"
                :id="`team-${teamIndex}-winner`"
                :value="teamIndex"
                v-model="winningTeam"
                name="winner"
                required
                class="mr-2"
              />
              <label :for="`team-${teamIndex}-winner`" class="text-sm font-medium text-gray-700"
                >Winner</label
              >
            </div>
          </div>
        </div>

        <div class="bg-white shadow-md rounded-lg p-6 mb-8">
          <h2 class="text-2xl font-semibold mb-4">Match Details</h2>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <label for="match-id" class="block text-sm font-medium text-gray-700">Match ID</label>
              <input
                type="text"
                id="match-id"
                v-model="matchId"
                required
                class="mt-1 block w-full border-gray-300 rounded-md border shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
              />
            </div>
            <div>
              <label for="match-length" class="block text-sm font-medium text-gray-700"
                >Match Length (minutes)</label
              >
              <input
                type="number"
                id="match-length"
                v-model.number="matchLength"
                required
                min="1"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
              />
            </div>
            <div>
              <label for="is-playoff" class="block text-sm font-medium text-gray-700"
                >Is Playoff</label
              >
              <select
                id="is-playoff"
                v-model="isPlayoff"
                required
                class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md"
              >
                <option value="">Select an option</option>
                <option :value="true">Yes</option>
                <option :value="false">No</option>
              </select>
            </div>
            <div>
              <label for="season" class="block text-sm font-medium text-gray-700">Season</label>
              <input
                type="number"
                id="season"
                v-model="season"
                required
                min="7"
                max="7"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
              />
            </div>
            <div>
              <label for="week" class="block text-sm font-medium text-gray-700">Week</label>
              <input
                type="number"
                id="week"
                v-model="week"
                required
                min="1"
                max="9"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
              />
            </div>
          </div>
        </div>

        <div class="flex justify-center space-x-4">
          <label
            for="file-upload"
            class="bg-gray-600 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline cursor-pointer"
          >
            Upload CSV
            <input
              id="file-upload"
              type="file"
              @change="handleFileUpload"
              class="hidden"
              accept=".csv"
            />
          </label>
          <button
            type="submit"
            class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
          >
            Generate CSV
          </button>
        </div>
      </form>

      <div v-if="errorMessage" class="mt-4 text-red-600 text-center">
        {{ errorMessage }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed } from 'vue'
import { supabase } from '../lib/supabaseClient.js'

const getRoleName = (index) => {
  const roles = ['Top', 'Jungle', 'Mid', 'Bot', 'Support']
  return roles[index]
}

const teams = reactive([
  {
    id: '',
    players: Array(5)
      .fill()
      .map(() => ({
        id: '',
        championId: '',
        kills: null,
        deaths: null,
        assists: null,
        cs: 0,
        isSub: false
      }))
  },
  {
    id: '',
    players: Array(5)
      .fill()
      .map(() => ({
        id: '',
        championId: '',
        kills: null,
        deaths: null,
        assists: null,
        cs: 0,
        isSub: false
      }))
  }
])

const winningTeam = ref(null)
const matchLength = ref(null)
const isPlayoff = ref(false)
const season = ref(7)
const week = ref(0)
const csvContent = ref('')
const errorMessage = ref('')

const teamOptions = ref([])
const playerOptions = ref([[], []])
const allPlayerOptions = ref([])
const championOptions = ref([])

// Match id should be team1name vs team2name _ reg if not playoff then week #
const matchId = computed(() => {
  if (teams[0].id && teams[1].id) {
    const team1 = teamOptions.value.find((team) => team.id === teams[0].id)
    const team2 = teamOptions.value.find((team) => team.id === teams[1].id)
    const formatString = (str) => str.toLowerCase().replace(/\s+/g, '_')
    return `${formatString(team1.name)}_vs_${formatString(team2.name)}_${isPlayoff.value ? 'playoff' : `week_${week.value}`}`
  }
  return ''
})

const roleMap = {
  0: 'TOP',
  1: 'JUNGLE',
  2: 'MIDDLE',
  3: 'BOTTOM',
  4: 'UTILITY'
}

const fetchTeams = async () => {
  const { data, error } = await supabase
    .from('teams')
    .select('team_id, name')
    .eq('season', season.value)

  return data
    ? data.map((team) => ({
        id: team.team_id,
        name: team.name
      }))
    : []
}

const fetchPlayers = async (teamId) => {
  const { data, error } = await supabase
    .from('team_players')
    .select(`player_id, team_id, teams(team_id), players(player_id, name)`)
    .eq('team_id', teamId)

  const players = data.map((player) => player.players)
  return data
    ? players.map((player) => ({
        id: player.player_id,
        name: player.name
      }))
    : []
}

const fetchChampions = async () => {
  const { data, error } = await supabase.from('champions').select('internal_name, name')

  return data
    ? data.map((champion) => ({
        id: champion.internal_name,
        name: champion.name
      }))
    : []
}

const loadTeams = async () => {
  try {
    teamOptions.value = await fetchTeams()
  } catch (error) {
    console.error('Error fetching teams:', error)
    errorMessage.value = 'Failed to load teams. Please try again.'
  }
}

const loadPlayers = async (teamIndex) => {
  const teamId = teams[teamIndex].id
  if (teamId) {
    try {
      playerOptions.value[teamIndex] = await fetchPlayers(teamId)
    } catch (error) {
      console.error('Error fetching players:', error)
      errorMessage.value = 'Failed to load players. Please try again.'
    }
  } else {
    playerOptions.value[teamIndex] = []
  }
}

const loadAllPlayers = async () => {
  try {
    const { data, error } = await supabase
      .from('players')
      .select('player_id, name')
      .order('name', { ascending: true }) // Alphabetical sort
      .order('created_at', { ascending: false }) // Then by created_at descending
    if (error) throw error

    allPlayerOptions.value = data.map((player) => ({
      id: player.player_id,
      name: player.name
    }))
  } catch (error) {
    console.error('Error fetching all players:', error)
    errorMessage.value = 'Failed to load all players. Please try again.'
  }
}

const loadChampions = async () => {
  try {
    championOptions.value = await fetchChampions()
  } catch (error) {
    console.error('Error fetching champions:', error)
    errorMessage.value = 'Failed to load champions. Please try again.'
  }
}

onMounted(async () => {
  await loadTeams()
  await loadChampions()
  await loadAllPlayers()
})

const handleSubChange = (teamIndex, playerIndex) => {
  const player = teams[teamIndex].players[playerIndex]
  if (player.isSub) {
    player.id = '' // Reset player selection when sub is checked
  }
}

const generateCSV = () => {
  errorMessage.value = ''

  const headers = [
    'player_id',
    'match_id',
    'team_id',
    'winning_team_id',
    'kills',
    'deaths',
    'assists',
    'cs',
    'champion_id',
    'individual_position',
    'match_length',
    'is_playoff',
    'season',
    'week',
    'team'
  ]

  const rows = teams.flatMap((team, teamIndex) =>
    team.players.map((player, playerIndex) => [
      player.id,
      matchId.value,
      team.id,
      winningTeam.value === teamIndex ? team.id : teams.find((t) => t.id !== team.id).id,
      player.kills,
      player.deaths,
      player.assists,
      player.cs,
      player.championId,
      roleMap[playerIndex],
      matchLength.value,
      isPlayoff.value,
      season.value,
      week.value,
      teamIndex === 0 ? 100 : 200
    ])
  )

  const csv = [headers.join(','), ...rows.map((row) => row.join(','))].join('\n')

  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  if (link.download !== undefined) {
    const url = URL.createObjectURL(blob)
    link.setAttribute('href', url)
    link.setAttribute('download', `match-${matchId.value}.csv`)
    link.style.visibility = 'hidden'
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
  }

  csvContent.value = csv
}

const parseCSV = async (content) => {
  const rows = content.split('\n').map((row) => row.split(','))
  const headers = rows.shift()

  teams.forEach((team) => {
    team.id = ''
    team.players.forEach((player) => {
      player.id = ''
      player.championId = ''
      player.kills = null
      player.deaths = null
      player.assists = null
      player.cs = null
      player.isSub = false
    })
  })

  const uniqueTeamIds = new Set()

  for (const row of rows) {
    if (row.length === headers.length) {
      const playerData = {}
      headers.forEach((header, index) => {
        playerData[header] = row[index]
      })

      uniqueTeamIds.add(playerData.team_id)

      const teamIndex = teams.findIndex((team) => team.id === playerData.team_id)
      const playerIndex = ['TOP', 'JUNGLE', 'MIDDLE', 'BOTTOM', 'UTILITY'].indexOf(
        playerData.individual_position
      )

      if (teamIndex === -1) {
        const newTeamIndex = teams.findIndex((team) => team.id === '')
        if (newTeamIndex !== -1) {
          teams[newTeamIndex].id = playerData.team_id
        }
      }

      const updatedTeamIndex = teams.findIndex((team) => team.id === playerData.team_id)
      if (updatedTeamIndex !== -1 && playerIndex !== -1) {
        teams[updatedTeamIndex].players[playerIndex] = {
          id: playerData.player_id,
          championId: playerData.champion_id,
          kills: parseInt(playerData.kills),
          deaths: parseInt(playerData.deaths),
          assists: parseInt(playerData.assists),
          cs: parseInt(playerData.cs),
          team: updatedTeamIndex === 0 ? 100 : 200,
          isSub: false // Set isSub to false when importing CSV
        }
      }

      if (playerData.winning_team_id) {
        winningTeam.value = teams.findIndex((team) => team.id === playerData.winning_team_id)
      }

      matchId.value = playerData.match_id
      matchLength.value = parseInt(playerData.match_length)
      isPlayoff.value = playerData.is_playoff.toLowerCase() === 'true'
      season.value = parseInt(playerData.season)
      week.value = parseInt(playerData.week)
    }
  }

  await loadTeamOptions(Array.from(uniqueTeamIds))
  for (const team of teams) {
    if (team.id) {
      await loadPlayerOptions(team.id, teams.indexOf(team))
    }
  }
}

const loadTeamOptions = async (teamIds) => {
  try {
    const { data, error } = await supabase
      .from('teams')
      .select('team_id, name')
      .in('team_id', teamIds)

    if (error) throw error

    teamOptions.value = data.map((team) => ({
      id: team.team_id,
      name: team.name
    }))
  } catch (error) {
    console.error('Error fetching teams:', error)
    errorMessage.value = 'Failed to load teams. Please try again.'
  }
}

const loadPlayerOptions = async (teamId, teamIndex) => {
  try {
    const { data, error } = await supabase
      .from('team_players')
      .select(`player_id, team_id, teams(team_id), players(player_id, name)`)
      .eq('team_id', teamId)

    if (error) throw error

    playerOptions.value[teamIndex] = data.map((item) => ({
      id: item.players.player_id,
      name: item.players.name
    }))
  } catch (error) {
    console.error('Error fetching players:', error)
    errorMessage.value = 'Failed to load players. Please try again.'
  }
}

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  const reader = new FileReader()

  reader.onload = async (e) => {
    const content = e.target.result
    csvContent.value = content
    await parseCSV(content)
  }

  reader.readAsText(file)
}
</script>

<style>
input:invalid,
select:invalid {
  border-color: #f56565;
}

input:invalid:focus,
select:invalid:focus {
  box-shadow: 0 0 0 3px rgba(245, 101, 101, 0.5);
}
</style>
