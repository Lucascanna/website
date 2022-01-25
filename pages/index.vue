<template>
  <!-- component -->
  <div class="font-mono bg-gray-400">
    <!-- Container -->
    <div class="container mx-auto">
      <div class="flex justify-center px-6 my-12">
        <!-- Row -->
        <div class="w-full xl:w-3/4 lg:w-11/12 flex">
          <!-- Col -->
          <div
            class="w-full h-auto bg-gray-400 hidden lg:block lg:w-5/12 bg-cover rounded-l-lg"
            style="background-image: url('https://source.unsplash.com/Mv9hjnEUHR4/600x800')"
          />
          <!-- Col -->
          <div class="w-full lg:w-7/12 bg-white p-5 rounded-lg lg:rounded-l-none">
            <h3 class="pt-4 text-2xl text-center">
              Reimbursebot
            </h3>
            <form class="px-8 pt-6 pb-8 mb-4 bg-white rounded">
              <!-- <div class="mb-4">
                <div class="mb-4 md:mr-2 md:mb-0">
                  <label class="block mb-2 text-sm font-bold text-gray-700" for="fullName">
                    Full name
                  </label>
                  <input
                    id="fullName"
                    class="w-full px-3 py-2 text-sm leading-tight text-gray-700 border rounded shadow appearance-none focus:outline-none focus:shadow-outline"
                    type="text"
                    placeholder="Full name"
                  >
                </div>
              </div>
              <div class="mb-4">
                <label class="block mb-2 text-sm font-bold text-gray-700" for="companyName">
                  Comapany Name
                </label>
                <input
                  id="companyName"
                  class="w-full px-3 py-2 mb-3 text-sm leading-tight text-gray-700 border rounded shadow appearance-none focus:outline-none focus:shadow-outline"
                  type="text"
                  placeholder="Company Name"
                >
              </div>
              <hr class="mb-6 border-t">
              <div class="mb-4 md:flex md:justify-between">
                <div class="mb-4 md:mr-2 md:mb-0">
                  <label class="block mb-2 text-sm font-bold text-gray-700" for="password">
                    Password
                  </label>
                  <input
                    id="password"
                    class="w-full px-3 py-2 mb-3 text-sm leading-tight text-gray-700 border border-red-500 rounded shadow appearance-none focus:outline-none focus:shadow-outline"
                    type="password"
                    placeholder="******************"
                  >
                  <p class="text-xs italic text-red-500">
                    Please choose a password.
                  </p>
                </div>
                <div class="md:ml-2">
                  <label class="block mb-2 text-sm font-bold text-gray-700" for="c_password">
                    Confirm Password
                  </label>
                  <input
                    id="c_password"
                    class="w-full px-3 py-2 mb-3 text-sm leading-tight text-gray-700 border rounded shadow appearance-none focus:outline-none focus:shadow-outline"
                    type="password"
                    placeholder="******************"
                  >
                </div>
              </div> -->
              <div class="mb-6 text-center">
                <button
                  class="w-full px-4 py-2 font-bold text-white bg-blue-500 rounded-full hover:bg-blue-700 focus:outline-none focus:shadow-outline"
                  type="button"
                  @click="submit()"
                >
                  Submit
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    const now = new Date()
    const currentMonth = now.getMonth()
    const currentYear = now.getFullYear()
    const capitalizeFirstLetter = string => string.charAt(0).toUpperCase() + string.slice(1)
    const currentMonthString = capitalizeFirstLetter(now.toLocaleDateString('it', { month: 'long' }))
    const firstDay = new Date(currentYear, currentMonth, 1)
    const lastDay = new Date(currentYear, currentMonth + 1, 0)
    const monthYear = `${currentMonthString} ${currentYear}`
    return {
      templateId: '1IYjJFOOSw_u7BQz3qQX0qe9hIGdJj6MelhoTrtx60Bs',
      fullName: 'Luca Scannapieco',
      company: 'MIA srl',
      numberOfAttachments: 0,
      expenses: [
        {
          description: 'Rimborso spese forfettario mensile per la telefonia mobile',
          amount: 15
        }
      ],
      lastDayOfMonth: lastDay.toLocaleDateString('it', { day: '2-digit', month: '2-digit', year: 'numeric' }),
      firstDayOfMonth: firstDay.toLocaleDateString('it', { day: '2-digit', month: '2-digit', year: 'numeric' }),
      monthYear,
      filename: `Nota Spese - ${monthYear}`
    }
  },
  methods: {
    async submit () {
      const token = this.$cookies.get('token')
      if (!token) {
        redirectToLoginPage()
        return
      }

      console.log('Creating file from template...')
      const driveUrl = `https://www.googleapis.com/drive/v3/files/${this.$data.templateId}/copy?access_token=${token}`
      const { data: fileData } = await this.$axios.post(driveUrl, { name: this.$data.filename })
      console.log(fileData)

      console.log('Getting newly created file info...')
      const sheetsGetUrl = `https://sheets.googleapis.com/v4/spreadsheets/${fileData.id}?access_token=${token}`
      const { data: spreadsheetData } = await this.$axios.get(sheetsGetUrl)
      console.log(spreadsheetData)

      console.log('Filling out the template the spreadsheet...')
      const { sheetId } = spreadsheetData.sheets[0].properties
      const extraInfoFillingDescriptor = [
        {
          updateCells: {
            start: {
              sheetId,
              rowIndex: 5,
              columnIndex: 1
            },
            rows: { values: [{ userEnteredValue: { stringValue: this.$data.lastDayOfMonth } }] },
            fields: 'userEnteredValue'
          }
        },
        {
          updateCells: {
            start: {
              sheetId,
              rowIndex: 7,
              columnIndex: 3
            },
            fields: 'userEnteredValue',
            rows: { values: [{ userEnteredValue: { stringValue: this.$data.fullName } }] }
          }
        },
        {
          updateCells: {
            start: {
              sheetId,
              rowIndex: 12,
              columnIndex: 6
            },
            fields: 'userEnteredValue',
            rows: { values: [{ userEnteredValue: { stringValue: this.$data.monthYear } }] }
          }
        },
        {
          updateCells: {
            start: {
              sheetId,
              rowIndex: 44,
              columnIndex: 1
            },
            fields: 'userEnteredValue',
            rows: { values: [{ userEnteredValue: { numberValue: this.$data.numberOfAttachments } }] }
          }
        },
        {
          updateCells: {
            start: {
              sheetId,
              rowIndex: 2,
              columnIndex: 4
            },
            fields: 'userEnteredValue',
            rows: { values: [{ userEnteredValue: { stringValue: this.$data.company } }] }
          }
        },
        {
          updateCells: {
            start: {
              sheetId,
              rowIndex: 47,
              columnIndex: 6
            },
            fields: 'userEnteredValue',
            rows: { values: [{ userEnteredValue: { stringValue: this.$data.company } }] }
          }
        }
      ]
      const expensesCount = this.$data.expenses.length
      const expensesFillingDescriptor = {
        updateCells: {
          range: {
            sheetId,
            startRowIndex: 15 + expensesCount,
            endRowIndex: 16 + expensesCount,
            startColumnIndex: 0,
            endColumnIndex: 7
          },
          rows: this.$data.expenses.map(({ description, amount }, index) => ({
            values: [
              { userEnteredValue: { stringValue: this.$data.firstDayOfMonth } },
              { userEnteredValue: { stringValue: description } },
              { userEnteredValue: { stringValue: '' } },
              { userEnteredValue: { stringValue: '' } },
              { userEnteredValue: { stringValue: '' } },
              { userEnteredValue: { stringValue: '' } },
              { userEnteredValue: { numberValue: amount } }
            ]
          })),
          fields: 'userEnteredValue'
        }
      }
      const sheetsUpdateUrl = `https://sheets.googleapis.com/v4/spreadsheets/${fileData.id}:batchUpdate?access_token=${token}`
      await this.$axios.post(sheetsUpdateUrl, {
        requests: [
          expensesFillingDescriptor,
          ...extraInfoFillingDescriptor
        ]
      })
      console.log('Spreadsheet filled out!')
    }
  }
}

const redirectToLoginPage = () => {
  const scopes = [
    'https://www.googleapis.com/auth/spreadsheets',
    'https://www.googleapis.com/auth/drive',
    'https://www.googleapis.com/auth/gmail.send'
  ].join(' ')
  const redirectUrlRaw = [
    'https://accounts.google.com',
    '/o/oauth2/v2/auth/oauthchooseaccount?',
    'client_id=273531667040-uisss7j5l6gs0fmn3hhdotnilu7e8u1n.apps.googleusercontent.com&',
    `redirect_uri=${window.location.protocol + '//' + window.location.host}/auth/google&`,
    'response_type=token&',
    `scope=${scopes}&`,
    'include_granted_scopes=true&',
    'state=pass-through value&',
    'flowName=GeneralOAuthFlow'
  ].join('')
  const redirectUrl = encodeURI(redirectUrlRaw)
  console.log(redirectUrl)
  window.location = redirectUrl
}
</script>
