<script module lang="ts">
  export function iso(date: Date) {
    const pad = (n: number) => (n < 10 ? '0' + n : n)
    return date.getFullYear() + '-' + pad(date.getMonth() + 1) + '-' + pad(date.getDate())
  }
</script>

<script lang="ts">
  import dayjs from 'dayjs'
  import { nextSVG, prevSVG } from ''

  let days = 'Mo|Tu|We|Th|Fr|Sa|Su'.split('|')
  let months = 'Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec'.split('|')
  let start = 1 // first day of the week (0 = Sunday, 1 = Monday)
  let offset1 = $state(0) // offset in months from currently selected date
  let offset2 = $state(0)

  let today = iso(new Date())


  let month1Date = new Date()
  let month2Date = dayjs().add(1, 'month').toDate()

  interface Props {
    date1: string | Date | undefined;
    date2: string | Date | undefined;
    background?: string;
    navClasses?: string;
    futureDatesOnly?: boolean;
    pastDatesOnly?: boolean;
    selectFirstDate?: boolean;
    ondateChange?: (date1: string | Date | undefined, date2: string | Date | undefined) => void
    onclick?: (e: MouseEvent & { currentTarget: EventTarget & HTMLElement }) => void
  }

  let {
    date1 = $bindable(),
    date2 = $bindable(),
    background = 'bg-white text-black dark:bg-surface-800 dark:text-white',
    navClasses = 'bg-transparent',
    futureDatesOnly = false,
    pastDatesOnly = false,
    selectFirstDate = $bindable(true),
    ondateChange = () => {},
    onclick = () => {},
  }: Props = $props();

  function changeMonth1(direction: number) {
    offset1 = offset1 + direction
  }

  function changeMonth2(direction: number) {
    offset2 = offset2 + direction
  }

  function monthIncrement(date: Date, offset: number, direction: number) {
    var dateWrap = new Date(date)
    dateWrap.setMonth(dateWrap.getMonth() + offset + direction)
    return dateWrap.getMonth()
  }

  function selectDate(newDate: string) {
    if (selectFirstDate) {
      date1 = newDate

      if (dayjs(date1).isAfter(date2)) {
        date2 = undefined
      }
    } else {
      if (dayjs(newDate).isBefore(date1)) {
        date1 = newDate
        date2 = undefined
        selectFirstDate = !selectFirstDate
      } else {
        date2 = newDate
      }
    }

    selectFirstDate = !selectFirstDate
    ondateChange(date1, date2)
  }

  let htmlMonth1: string = $state() as string
  let htmlMonth2: string = $state() as string
  let htmlYear1: number = $state() as number
  let htmlYear2: number = $state() as number






// m and y are for reactivity
  function syncMonths(m1: any, m2: any) {
    htmlMonth1 = month1
    htmlMonth2 = month2
  }

  function syncYears(y1: any, y2: any) {
    htmlYear1 = year1
    htmlYear2 = year2
  }

  function monthChange1(newMonth: string) {
    changeMonth1(months.findIndex((item) => item === newMonth) - viewDate1.getMonth())
  }

  function yearChange1(newYear: number) {
    changeMonth1((newYear - viewDate1.getFullYear()) * 12)
  }

  function monthChange2(newMonth: string) {
    changeMonth2(months.findIndex((item) => item === newMonth) - viewDate2.getMonth())
  }

  function yearChange2(newYear: number) {
    changeMonth2((newYear - viewDate2.getFullYear()) * 12)
  }

  function viewDateFrom(date: string | number | Date, offset: number) {
    var viewDate = new Date(date)
    viewDate.setMonth(viewDate.getMonth() + offset)
    return viewDate
  }

  function weeksFrom(
    viewDate: Date,
    date1: string | Date | undefined,
    date2: string | Date | undefined,
    start: number
  ) {
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

    const isBeforeToday = (date: Date | string) => dayjs(date).isBefore(today)
    const isAfterToday = (date: Date | string) => dayjs(date).isAfter(today)
    const isSameDay = (date1: Date | string, date2: Date | string) =>
      dayjs(date1).startOf('day').isSame(dayjs(date2).startOf('day'))
    const isWithinRange = (date: Date | string, start: Date | string, end: Date | string) =>
      (dayjs(date).isAfter(start, 'day') && dayjs(date).isBefore(end, 'day')) ||
      isSameDay(date, start) ||
      isSameDay(date, end)
    const isOutsideRange = (date: Date | string, start: Date | string, end: Date | string) =>
      !isWithinRange(date, start, end) && !isSameDay(date, start) && !isSameDay(date, end)

    while (d.getTime() <= last.getTime()) {
      var dd = d.getDate(),
        mm = d.getMonth(),
        yy = d.getFullYear(),
        value = iso(d)

      let isDayFromThisMonth = mm == M

      week.push({
        date: dd,
        value,
        class: [
          (futureDatesOnly && isBeforeToday(value)) || (pastDatesOnly && isAfterToday(value))
            ? 'opacity-40 hover:bg-primary-300/20 hover:cursor-default'
            : isDayFromThisMonth && date1 && isSameDay(value, date1)
              ? 'selected text-white bg-primary-500 font-bold hover:bg-primary-600 rounded-s-xl'
              : isDayFromThisMonth && date2 && isSameDay(value, date2)
                ? 'selected text-white bg-primary-500 font-bold hover:bg-primary-600 rounded-e-xl'
                : date1 && date2 && isOutsideRange(value, date1, date2)
                  ? 'hover:bg-gray-300 dark:hover:bg-gray-600 hover:rounded-lg'
                  : date1 && date2 && isWithinRange(value, date1, date2)
                    ? 'bg-gray-200 text-black dark:bg-gray-700 dark:text-white hover:bg-gray-300 dark:hover:bg-gray-600'
                    : '',
          isDayFromThisMonth
            ? ''
            : (mm > M ? yy >= Y : yy > Y)
              ? 'future opacity-40'
              : 'past opacity-40'
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
  let viewDate1 = $derived(viewDateFrom(month1Date, offset1))
  let viewDate2 = $derived(viewDateFrom(month2Date, offset2))
  let month1 = $derived(months[viewDate1.getMonth()])
  let month2 = $derived(months[viewDate2.getMonth()])
  let year1 = $derived(viewDate1.getFullYear())
  let year2 = $derived(viewDate2.getFullYear())
  let weeks1 = $derived(weeksFrom(viewDate1, date1, date2, start))
  let weeks2 = $derived(weeksFrom(viewDate2, date1, date2, start))

  $effect(() => {
    syncMonths(month1, month2)
  });

  $effect(() => {
    syncYears(year1, year2)
  });
</script>

<div class="flex flex-row w-fit z-50">
  <div class="flex flex-col">
    <div class="month-year flex flex-row justify-between justify-items-center w-fit">
      <button
        class="mr-2 padding-left: 8px cursor-pointer"
        onclick={(e) => {
          e.stopPropagation()
          changeMonth1(-1)}
          }>{@html prevSVG}</button
      >
      <div class="flex flex-row">
        <select
          onclick={(e) => {
            e.stopPropagation();
            onclick(e)}
          }
          class="{navClasses} {background} focus:outline-none border-0 focus:ring-0 pl-0 ml-2 cursor-pointer"
          bind:value={htmlMonth1}
          onchange={(e) => monthChange1(htmlMonth1)}
        >
          {#each months as month}
            <option
              value={month}
              disabled={year1 === year2 &&
                months.findIndex((item) => item === month) >=
                  months.findIndex((item) => item === month2)}>{month}</option
            >
          {/each}
        </select>
        <select
          onclick={(e) => {
            e.stopPropagation();
            onclick(e)}
          }
          class="{navClasses} {background} focus:outline-none border-0 focus:ring-0 pl-0 cursor-pointer"
          bind:value={htmlYear1}
          onchange={(e) => yearChange1(htmlYear1)}
        >
          {#each Array(10) as _, i}
            {#if futureDatesOnly}
              <option
                value={dayjs(today).get('year') + i}
                disabled={dayjs(today).get('year') + i > year2 ||
                  (dayjs(today).get('year') + i === year2 &&
                    months.findIndex((item) => item === month1) >=
                      months.findIndex((item) => item === month2))}
                >{dayjs(today).get('year') + i}</option
              >
            {:else if pastDatesOnly}
              <option
                value={dayjs(today).get('year') - i}
                disabled={dayjs(today).get('year') - i > year2 ||
                  (dayjs(today).get('year') - i === year2 &&
                    months.findIndex((item) => item === month1) >=
                      months.findIndex((item) => item === month2))}
                >{dayjs(today).get('year') - i}</option
              >
            {:else}
              <option
                value={dayjs(today).get('year') - 5 + i}
                disabled={dayjs(today).get('year') - 5 + i > year2 ||
                  (dayjs(today).get('year') - 5 + i === year2 &&
                    months.findIndex((item) => item === month1) >=
                      months.findIndex((item) => item === month2))}
                >{dayjs(today).get('year') - 5 + i}</option
              >
            {/if}
          {/each}
        </select>
      </div>
      <button
        class="ml-2 cursor-pointer"
        disabled={dayjs(month1Date)
          .add(offset1 + 1, 'month')
          .isSame(dayjs(month2Date).add(offset2, 'month'))}
        onclick={(e) => {
          e.stopPropagation();
          changeMonth1(+1)}
        }>{@html nextSVG}</button
      >
    </div>

    <div class="date-picker flex justify-center">
      <table>
        <thead>
          <tr>
            {#each days as day}
              <th class="font-medium">{day}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each weeks1 as week}
            <tr>
              {#each week as day}
                <td
                  class="btn-date {day.class} h-8 w-8 text-sm text-center hover:cursor-pointer"
                  onclick={(e) => {
                    if (
                      (!futureDatesOnly || !dayjs(day.value).isBefore(today)) &&
                      (!pastDatesOnly || !dayjs(day.value).isAfter(today))
                    ) {
                      selectDate(day.value)
                    } else {
                      e.stopPropagation()
                    }
                  }}>{day.date}</td
                >
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>

  <div class="ml-5 flex flex-col">
    <div class="month-year flex flex-row justify-between justify-items-center w-fit">
      <button
        class="mr-2 padding-left: 8px cursor-pointer"
        disabled={dayjs(month2Date)
          .add(offset2 - 1, 'month')
          .isSame(dayjs(month1Date).add(offset1, 'month'))}
        onclick={(e) => {
          e.stopPropagation();
          changeMonth2(-1)}
        }>{@html prevSVG}</button
      >
      <div class="flex flex-row">
        <select
          onclick={(e) => {
            e.stopPropagation();
            onclick(e)}
          }
          class="{navClasses} {background} focus:outline-none border-0 focus:ring-0 pl-0 ml-2 cursor-pointer"
          bind:value={htmlMonth2}
          onchange={(e) => monthChange2(htmlMonth2)}
        >
          {#each months as month}
            <option
              value={month}
              disabled={year1 === year2 &&
                months.findIndex((item) => item === month) <=
                  months.findIndex((item) => item === month1)}>{month}</option
            >
          {/each}
        </select>
        <select
          onclick={(e) => {
            e.stopPropagation()
            onclick(e)}
          }
          class="{navClasses} {background} focus:outline-none border-0 focus:ring-0 pl-0 cursor-pointer"
          bind:value={htmlYear2}
          onchange={(e) => yearChange2(htmlYear2)}
        >
          {#each Array(10) as _, i}
            {#if futureDatesOnly}
              <option
                value={dayjs(today).get('year') + i}
                disabled={dayjs(today).get('year') + i < year1 ||
                  (dayjs(today).get('year') + i === year1 &&
                    months.findIndex((item) => item === month2) <=
                      months.findIndex((item) => item === month1))}
                >{dayjs(today).get('year') + i}</option
              >
            {:else if pastDatesOnly}
              <option
                value={dayjs(today).get('year') - i}
                disabled={dayjs(today).get('year') - i < year1 ||
                  (dayjs(today).get('year') - i === year1 &&
                    months.findIndex((item) => item === month2) <=
                      months.findIndex((item) => item === month1))}
                >{dayjs(today).get('year') - i}</option
              >
            {:else}
              <option
                value={dayjs(today).get('year') - 5 + i}
                disabled={dayjs(today).get('year') - 5 + i < year1 ||
                  (dayjs(today).get('year') - 5 + i === year1 &&
                    months.findIndex((item) => item === month2) <=
                      months.findIndex((item) => item === month1))}
                >{dayjs(today).get('year') - 5 + i}</option
              >
            {/if}
          {/each}
        </select>
      </div>
      <button class="ml-2 cursor-pointer" onclick={(e) => {
        e.stopPropagation();
        changeMonth2(+1)}
      }
        >{@html nextSVG}</button
      >
    </div>

    <div class="date-picker flex justify-center">
      <table>
        <thead>
          <tr>
            {#each days as day}
              <th class="font-medium">{day}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each weeks2 as week}
            <tr>
              {#each week as day}
                <td
                  class="btn-date {day.class} h-8 w-8 text-sm text-center hover:cursor-pointer"
                  onclick={(e) => {
                    if (
                      (!futureDatesOnly || !dayjs(day.value).isBefore(today)) &&
                      (!pastDatesOnly || !dayjs(day.value).isAfter(today))
                    ) {
                      selectDate(day.value)
                    } else {
                      e.stopPropagation()
                    }
                  }}>{day.date}</td
                >
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</div>
