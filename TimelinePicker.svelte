<!-- UNFINISHED -->
<script lang="ts">
    import dayjs from "dayjs"

    interface Props {
        colour?: string;
        workingHours: {open_time: string, close_time: string};
        times: {start: Date, end: Date}[];
    }

    let { colour = "secondary-500", workingHours, times }: Props = $props();

    let timeline: HTMLDivElement = $state() as HTMLDivElement

    function insertNewTime(time: string) {
        let xCoordinate = 
            dayjs(`01-01-2000 ${time}`).diff(dayjs(`01-01-2000 ${workingHours.open_time}`)) /
            dayjs(`01-01-2000 ${workingHours.open_time}`).diff(dayjs(`01-01-2000 ${workingHours.close_time}`)) * 
            timeline.offsetWidth
        
        insertNewDot(xCoordinate)
    }

    function insertNewDot(xCoordinate: number) {

        let newDot = document.createElement('div')
        newDot.style.width = "0.75rem"
        newDot.style.height = "0.75rem"
        newDot.style.marginLeft = `${xCoordinate-2}px`
        newDot.style.position = "absolute"
        newDot.style.backgroundColor = colour
        newDot.style.borderRadius = "100%"
        newDot.style.backgroundColor = `rgba(var(--color-${colour})/1)`
        document.getElementById("timelinepicker")?.appendChild(newDot)
    }

    $effect(() => {
        times && timeline && times.forEach(time => {insertNewTime(dayjs(time.start).format('HH:mm')); insertNewTime(dayjs(time.end).format('HH:mm'))})
    });
</script>

<div id="timelinepicker" class="flex flex-row ml-4 mr-4">
    <div class="w-3 h-3 rounded-full absolute bg-{colour}"></div>
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div class="timeline-wrap w-full h-5 hover:cursor-pointer"
        onclick={(e) => insertNewDot(e.offsetX-3)}>
        <div bind:this={timeline} class="timeline w-full h-1 mt-1 bg-{colour}"
></div>
    </div>
    <div class="w-3 h-3 relative rounded-full bg-{colour}"></div>
</div>