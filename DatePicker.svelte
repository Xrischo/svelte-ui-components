<script module lang="ts">
  export function iso(date: Date) {
    const pad = (n: number) => (n < 10 ? '0' + n : n)
    return date.getFullYear() + '-' + pad(date.getMonth() + 1) + '-' + pad(date.getDate())
  }
</script>

<script lang="ts">
  import dayjs from 'dayjs'
  import { nextSVG, nextSVG2, prevSVG, prevSVG2 } from ''


  let today = iso(new Date())
  let days = 'Mo|Tu|We|Th|Fr|Sa|Su'.split('|')
  let months = 'Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec'.split('|')
  interface Props {
    date?: string | Date | null;
    start?: number;
    offset?: number;
    size?: 'small' | 'medium';
    padding?: string;
    navClasses?: string;
    futureDatesOnly?: boolean;
    pastDatesOnly?: boolean;
    restrictedYears?: boolean;
    background?: string;
    z_value?: string;
    ondateChange?: (date: string) => void
    onclick?: (e: MouseEvent & { currentTarget: EventTarget & HTMLSelectElement }) => void
  }

  let {
    date = $bindable(iso(new Date())),
    start = 1,
    offset = $bindable(0),
    size = 'medium',
    padding = '',
    navClasses = 'bg-transparent',
    futureDatesOnly = false,
    pastDatesOnly = false,
    restrictedYears = true,
    background = 'bg-surface-100 text-black dark:bg-surface-700 dark:text-white',
    z_value = 'z-[100]',
    ondateChange = () => {},
    onclick = () => {}
  }: Props = $props();

  function go(direction: number) {
    offset = offset + direction
  }

  function selectDate(newDate: string) {
    date = newDate
    offset = 0
    ondateChange(date)
  }

  function monthChange(newMonth: string) {
    go(months.findIndex((item) => item === newMonth) - viewDate.getMonth())
  }

  function yearChange(newYear: number) {
    go((newYear - viewDate.getFullYear()) * 12)
  }

  let htmlMonth: string = $state() as string
  let htmlYear: number = $state() as number

  // m and y are for reactivity
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

  function weeksFrom(viewDate: Date, date: string | Date, start: number) {
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
            : pastDatesOnly && dayjs(value).isAfter(today)
            ? 'opacity-40 hover:bg-primary-300/20 hover:cursor-default'
            : value === date
            ? 'selected text-white bg-primary-500 font-bold rounded-xl hover:bg-primary-600 hover:cursor-pointer'
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
  let viewDate = $derived(date ? viewDateFrom(date, offset) : viewDateFrom(today, offset))
  let month = $derived(months[viewDate.getMonth()])
  let year = $derived(viewDate.getFullYear())
  let weeks = $derived(date ? weeksFrom(viewDate, date, start) : weeksFrom(viewDate, today, start))
  $effect(() => {
    syncMonths(month)
  });
  $effect(() => {
    syncYears(year)
  });
</script>

<div class="flex flex-col w-fit {z_value} {padding}">
  <div
    class="month-year w-full flex flex-row justify-center justify-between justify-items-center w-fit"
  >
    <button
      class="mr-2"
      style={size === 'small' ? 'padding-left: 8px;' : ''}
      onclick={(e) => {e.stopPropagation(); go(-1)}}>{@html prevSVG}</button
    >
    <div class="flex flex-row">
      <select
        onclick={(e) => {e.stopPropagation(); onclick(e)}}
        class="{navClasses} {background} focus:outline-none border-0 focus:ring-0 pl-0 ml-2 hover:cursor-pointer"
        bind:value={htmlMonth}
        onchange={(e) => monthChange(htmlMonth)}
      >
        {#each months as month}
          <option value={month}>{month}</option>
        {/each}
      </select>
      <select
        onclick={(e) => {e.stopPropagation(); onclick(e)}}
        class="{navClasses} {background} focus:outline-none border-0 focus:ring-0 pl-0 hover:cursor-pointer"
        bind:value={htmlYear}
        onchange={(e) => yearChange(htmlYear)}
      >
        {#each restrictedYears ? Array(10) : Array(100) as _, i}
          {#if futureDatesOnly}
            <option value={dayjs(today).get('year') + i}>{dayjs(today).get('year') + i}</option>
          {:else if pastDatesOnly}
            <option value={dayjs(today).get('year') - i}>{dayjs(today).get('year') - i}</option>
          {:else}
            <option value={dayjs(today).get('year') - 5 + i}
              >{dayjs(today).get('year') - 5 + i}</option
            >
          {/if}
        {/each}
      </select>
    </div>
    <button class="ml-2" onclick={(e) => {e.stopPropagation(); go(+1)}}>{@html nextSVG}</button>
  </div>

  <div class="date-picker {size === 'small' ? '' : 'mt-5'} flex justify-center">
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
                (!futureDatesOnly || !dayjs(day.value).isBefore(today)) &&
                (!pastDatesOnly || !dayjs(day.value).isAfter(today))
                  ? selectDate(day.value)
                  : e.stopPropagation()}>{day.date}</td
            >
          {/each}
        </tr>
        </tbody>
      {/each}
    </table>
  </div>

  <div class="flex flex-row justify-between">
    <button
      disabled={!(
        !futureDatesOnly ||
        (futureDatesOnly && dayjs(date).startOf('day').isAfter(dayjs(today).startOf('day')))
      )}
      class="btn"
      onclick={(e) => {
        e.stopPropagation();
        date = dayjs(date).subtract(1, 'day').format('YYYY-MM-DD')
      }}>{@html prevSVG2}</button
    >
    <button
      class="btn text-sm"
      onclick={(e) => {
        e.stopPropagation();
        date = dayjs().format('YYYY-MM-DD')
      }}>
      TODAY
    </button>
    <button
      disabled={!(
        !pastDatesOnly ||
        (pastDatesOnly && dayjs(date).endOf('day').isBefore(dayjs(today).endOf('day')))
      )}
      class="btn"
      onclick={(e) => {
        e.stopPropagation();
        date = dayjs(date).add(1, 'day').format('YYYY-MM-DD')
      }}>{@html nextSVG2}</button
    >
  </div>
</div>
