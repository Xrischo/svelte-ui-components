<script module lang="ts">
  export function iso(date: Date) {
    const pad = (n: number) => (n < 10 ? '0' + n : n)
    return date.getFullYear() + '-' + pad(date.getMonth() + 1) + '-' + pad(date.getDate())
  }
</script>

<script lang="ts">
  import dayjs from 'dayjs'
  import { nextSVG, prevSVG } from ''

  let today = iso(new Date())
  let lastDate = $state(today)
  let days = 'Mo|Tu|We|Th|Fr|Sa|Su'.split('|')
  let months = 'Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec'.split('|')
  interface Props {
    dates?: string[] | null
    start?: number
    offset?: number
    size?: 'small' | 'medium'
    padding?: string
    navClasses?: string
    futureDatesOnly?: boolean
    background?: string
    border?: string
    onclick?: (e: MouseEvent & { currentTarget: EventTarget & HTMLDivElement }) => void
  }

  let {
    dates = $bindable([]),
    start = 1,
    offset = $bindable(0),
    size = 'medium',
    padding = '',
    navClasses = 'bg-transparent',
    futureDatesOnly = false,
    background = 'bg-surface-100 text-black dark:bg-surface-700 dark:text-white',
    border = '',
    onclick = () => {}
  }: Props = $props()

  function go(direction: number) {
    offset = offset + direction
  }

  function selectDate(newDate: string) {
    if (!dates) dates = []

    dates = dates.includes(newDate) ? dates.filter((item) => item !== newDate) : [...dates, newDate]
    offset = 0
    // this bs is needed bcs for whatever reason if the two strings are equal, even reassignment doesn't trigger reactivity
    if (lastDate === newDate) {
      lastDate = lastDate + 'T00:00:00.000Z'
    } else {
      lastDate = newDate
    }
  }

  function monthChange(newMonth: string) {
    go(months.findIndex((item) => item === newMonth) - viewDate.getMonth())
  }

  function yearChange(newYear: number) {
    go((newYear - viewDate.getFullYear()) * 12)
  }

  let htmlMonth: string = $state() as string
  let htmlYear: number = $state() as number

  // m and y for reactivity
  function syncMonths(m: any) {
    htmlMonth = month
  }

  function syncYears(y: any) {
    htmlYear = year
  }

  function viewDateFrom(date: string | number | Date, offset: number) {
    var viewDate = new Date(date)
    viewDate.setDate(1)
    viewDate.setMonth(viewDate.getMonth() + offset)
    return viewDate
  }

  function weeksFrom(viewDate: Date, date: string, start: number) {
    var first = new Date(viewDate.getTime())
    first.setDate(1)
    first.setDate(first.getDate() - first.getDay() + 1)

    var last = new Date(viewDate.getTime())
    last.setDate(new Date(viewDate.getFullYear(), viewDate.getMonth() + 1, 0).getDate())
    last.setDate(last.getDate() + (7 - last.getDay()))

    var d = new Date(first.getTime()),
      M = viewDate.getMonth(),
      Y = viewDate.getFullYear(),
      week: { date: number; value: string; class: string }[] = [],
      weeks = [week]

    while (d.getTime() <= last.getTime()) {
      var dd = d.getDate(),
        mm = d.getMonth(),
        yy = d.getFullYear(),
        value = iso(d)

      week.push({
        date: dd,
        value,
        class: [
          futureDatesOnly && dayjs(value).isBefore(today)
            ? 'opacity-40 hover:bg-primary-300/20 hover:cursor-default'
            : dates?.includes(value)
              ? `selected text-white bg-primary-500 font-bold rounded-xl hover:bg-primary-600 hover:cursor-pointer`
              : value === today
                ? 'text-white bg-primary-400 rounded-xl hover:bg-primary-400/50 hover:cursor-pointer'
                : 'hover:bg-primary-300/20 hover:cursor-pointer',
          mm == M ? '' : (mm > M ? yy >= Y : yy > Y) ? 'future opacity-40' : 'past opacity-40'
        ].join(' ')
      })

      d = new Date(d.getFullYear(), d.getMonth(), d.getDate() + 1)

      if (d.getDay() === start) {
        week = []
        weeks.push(week)
      }
    }

    return weeks
  }
  let viewDate = $derived(viewDateFrom(lastDate, offset))
  let month = $derived(months[viewDate.getMonth()])
  let year = $derived(viewDate.getFullYear())
  let weeks = $derived(weeksFrom(viewDate, lastDate, start))
  $effect(() => {
    syncMonths(month)
  })
  $effect(() => {
    syncYears(year)
  })
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
  onclick={(e) => {
    e.stopPropagation()
    onclick(e)
  }}
  class="flex flex-col w-fit z-50 {padding}"
>
  <div class="month-year flex flex-row justify-between justify-items-center w-fit">
    <button class="mr-2" style={size === 'small' ? 'padding-left: 8px;' : ''} onclick={() => go(-1)}
      >{@html prevSVG}</button
    >
    <div class="flex flex-row">
      <select
        class="{navClasses} focus:outline-none border-0 focus:ring-0 pl-0 ml-2"
        bind:value={htmlMonth}
        onchange={(e) => monthChange(htmlMonth)}
      >
        {#each months as month}
          <option value={month}>{month}</option>
        {/each}
      </select>
      <select
        class="{navClasses} focus:outline-none border-0 focus:ring-0 pl-0"
        bind:value={htmlYear}
        onchange={(e) => yearChange(htmlYear)}
      >
        {#each Array(10) as _, i}
          {#if futureDatesOnly}
            <option value={dayjs(today).get('year') + i}>{dayjs(today).get('year') + i}</option>
          {:else}
            <option value={dayjs(today).get('year') - 5 + i}
              >{dayjs(today).get('year') - 5 + i}</option
            >
          {/if}
        {/each}
      </select>
    </div>
    <button class="ml-2" onclick={(e) => go(+1)}>{@html nextSVG}</button>
  </div>

  <div class="date-picker {size === 'small' ? '' : 'mt-5'} flex justify-center mb-2">
    <table>
      <thead>
        <tr>
          {#each days as day}
            <th class={size === 'small' ? 'font-medium' : ''}>{day}</th>
          {/each}
        </tr>
      </thead>
      {#each weeks as week}
        <tbody>
          <tr>
            {#each week as day}
              <td
                class="btn-date {day.class} {size === 'small'
                  ? 'h-8 w-8 text-sm'
                  : 'h-10 w-10'} text-center hover:rounded-xl"
                onclick={(e) =>
                  !futureDatesOnly || !dayjs(day.value).isBefore(today)
                    ? selectDate(day.value)
                    : e.stopPropagation()}>{day.date}</td
              >
            {/each}
          </tr>
        </tbody>
      {/each}
    </table>
  </div>
</div>
