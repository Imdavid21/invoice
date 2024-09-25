<script>
  import { Plus, Trash, ImagePlus, RotateCw, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '007',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [{ desc: '', price: '', quantity: '' }],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: ''
    },
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar' },
    { code: 'EUR', symbol: '€', name: 'Euro' },
    { code: 'GBP', symbol: '£', name: 'British Pound' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar' }
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  // Utils --------------------------------------------------------------

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (
      file &&
      (file.type === 'image/png' || file.type === 'image/jpeg' || file.type === 'image/webp')
    ) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [...appState.items, { desc: itemDesc, price: parseFloat(itemPrice).toFixed(2), quantity: parseFloat(itemQty).toFixed(2) }];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Wayne Enterprises', logo: '' },
      invoice: {
        number: '007',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Enter Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Enter Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [
        { desc: 'Batmobile Repair', price: '15000', quantity: '1' },
        { desc: 'Grappling Hooks', price: '500', quantity: '20' },
        { desc: 'Interdimensional Portal Gun', price: '200000', quantity: '1' }
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: ''
      },
      currency: 'USD'
    };

    save();
  }

  // Startup ------------------------------------------------------------

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    } else {
      reset();
    }
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  $: subTotal = appState.items.reduce((total, item) => {
    return total + item.price * item.quantity;
  }, 0).toFixed(2);

  $: discountAmount = (subTotal * (+appState.discountPercent || 0) / 100).toFixed(2);
  $: taxableAmount = (subTotal - discountAmount).toFixed(2);
  $: taxAmount = (taxableAmount * (+appState.taxPercent || 0) / 100).toFixed(2);
  $: totalDue = (parseFloat(taxableAmount) + parseFloat(taxAmount)).toFixed(2);

  // Toggles
  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<svelte:head>
  <title>Billie - No Strings Attached Invoice Generator</title>
  <meta
    name="description"
    content="Create and download invoices instantly, with zero signups or tracking."
  />
</svelte:head>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-montserrat bg-[#FAFAFA] border border-[#E2E2E2] rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- Company & Invoice Details --------------------------------------->
  <div class="flex flex-row justify-between items-start print:flex">
    <div class="flex flex-col gap-4">
      <div class="">
        {#if appState.company.logo}
          <div class="flex flex-row items-center space-x-4">
            <div class="flex flex-col border border-[#E2E2E2] rounded-xl">
              <button
                class="p-2 hover:bg-[#E2E2E2] rounded"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                <Trash />
              </button>
              <button class="p-2 hover:bg-[#E2E2E2] rounded" on:click={() => document.getElementById('imageInput').click()}>
                <ImagePlus />
                <input
                  id="imageInput"
                  type="file"
                  accept=".png,.jpg,.jpeg,.webp"
                  on:change={handleImageChange}
                  class="hidden"
                />
              </button>
            </div>
            <img
              src={appState.company.logo}
              alt="Company Logo"
              class="max-h-18 max-w-40 object-cover m-0"
            />
          </div>
        {:else}
          <button
            class="p-2 flex flex-row gap-2 rounded-lg border border-[#E2E2E2] cursor-pointer hover:bg-[#F5F5F5]"
            on:click={() => document.getElementById('imageInput').click()}
          >
            <ImagePlus />
            <p class="text-sm text-[#333]">Click to select an image for logo</p>
            <input
              id="imageInput"
              type="file"
              accept=".png,.jpg,.jpeg,.webp"
              on:change={handleImageChange}
              class="hidden"
            />
          </button>
        {/if}
      </div>
      <input class="font-bold text-xl border border-[#E2E2E2] p-3 rounded-lg focus:outline-none focus:border-[#5A5A5A]" type="text" bind:value={appState.company.name} />
    </div>

    <div class="relative flex flex-col items-end gap-2 print:items-start print:w-full">
      <h2 class="font-bold text-2xl text-[#1E6F5C] text-right print:text-left">
        INVOICE :
        <input
          type="text"
          size="4"
          placeholder="007"
          maxlength="4"
          bind:value={appState.invoice.number}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </h2>
      <button
        on:click={() => reset()}
        class="p-2 rounded-lg hover:bg-[#E2E2E2] transition-all duration-100 ease-in-out print:hidden"
      >
        <RotateCw strokeWidth={1.5} />
      </button>
      <div class="flex flex-col gap-1">
        <p>
          Created :
          <input
            bind:value={appState.invoice.created}
            type="text"
            size="10"
            placeholder="Date Created"
            maxlength="13"
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          />
        </p>
        <p>
          Due :
          <input
            bind:value={appState.invoice.due}
            type="text"
            size="10"
            placeholder="Due Date"
            maxlength="13"
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          />
        </p>
      </div>
      <select bind:value={appState.currency} class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]">
        {#each currencyOptions as option}
          <option value={option.code}>{option.code} - {option.name}</option>
        {/each}
      </select>
    </div>
  </div>

  <hr class="border-[#E2E2E2]" />

  <!-- From & To  ------------------------------------------------------>

  <div class="flex flex-row justify-between">
    <div class="flex flex-col gap-2">
      <h4 class="mb-2 font-bold">From</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Enter sender's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
      ></textarea>
      <div class="grid grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24">Tax Name:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxIdName}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Tax Name"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Tax ID:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Tax ID"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Email"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Phone"
          />
        </p>
      </div>
    </div>
    <div class="flex flex-col gap-2">
      <h4 class="mb-2 font-bold">To</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Enter recipient's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
      ></textarea>
      <div class="grid grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24">Tax Name:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxIdName}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Tax Name"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Tax ID:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Tax ID"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Email"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] w-full"
            placeholder="Enter Phone"
          />
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form --------------------------------------------------->

  <form class="flex flex-row justify-between gap-2">
    <button
      disabled={itemDesc == ''}
      on:click={() => addItem()}
      class="p-2 rounded-full bg-[#E2E2E2] hover:bg-[#CFCFCF] transition-all duration-100 ease-in-out print:hidden"
    >
      <Plus stroke="#557571" />
    </button>

    <input
      id="descBox"
      type="text"
      bind:value={itemDesc}
      placeholder="Item description"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] grow"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemPrice}
      placeholder="Unit Price"
      min="0"
      step="0.01"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemQty}
      placeholder="Quantity"
      min="1"
      step="1"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <p class="border border-[#E2E2E2] p-3 w-32 text-right">
      {getCurrencySymbol(appState.currency)}{(itemPrice * itemQty).toFixed(2)}
    </p>
  </form>

  <!-- Table ----------------------------------------------------------->

  <div class="flex flex-col mt-4">
    <div class="flex flex-row bg-[#FAFAFA] border border-[#E2E2E2]">
      <p class="font-bold p-3 border-r border-[#E2E2E2] grow">Item Description</p>
      <p class="font-bold p-3 border-r border-[#E2E2E2] w-32">Unit Price</p>
      <p class="font-bold p-3 border-r border-[#E2E2E2] w-24">Qty</p>
      <p class="font-bold p-3 w-32 text-right">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-row even:bg-[#F8F8F8] border border-[#E2E2E2]">
        <button
          on:click={() => deleteItem(index)}
          class="p-3 hover:bg-[#E2E2E2] transition-all duration-100 ease-in-out rounded print:hidden"
        >
          <Trash color="#C96868" />
        </button>

        <p contenteditable="true" class="p-3 border-r border-[#E2E2E2] grow">{item.desc}</p>
        <input class="p-3 border-r border-[#E2E2E2] w-32" type="text" bind:value={item.price} />
        <input class="p-3 border-r border-[#E2E2E2] w-24" type="text" bind:value={item.quantity} />
        <p class="p-3 w-32 text-right">{getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}</p>
      </div>
    {/each}

    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2] items-center">
      <button
        class="toggle-switch ml-2 mr-1"
        on:click={() => (isDiscountEnabled = !isDiscountEnabled)}
      ></button>
      <p class="p-3 grow text-right font-bold">Discount %</p>
      <input
        class="p-3 w-32 text-right"
        type="text"
        bind:value={appState.discountPercent}
        disabled={!isDiscountEnabled}
        step="0.01"
        max="100"
        min="0"
      />
    </div>
    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2] items-center">
      <button class="toggle-switch ml-2 mr-1" on:click={() => (isTaxEnabled = !isTaxEnabled)}></button>
      <p class="p-3 grow text-right font-bold">Tax %</p>
      <input class="p-3 w-32 text-right" type="text" bind:value={appState.taxPercent} disabled={!isTaxEnabled} step="0.01" max="100" min="0" />
    </div>
    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2]">
      <p class="p-3 grow text-right font-bold">Total Due</p>
      <p class="p-3 w-32 text-right">{getCurrencySymbol(appState.currency)}{totalDue}</p>
    </div>
  </div>

  <!-- Payment Info Section Redesigned --------------------------------->
  <div class="mt-4">
    <h4 class="mb-2 font-bold">Payment Info</h4>
    <div class="grid grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account</label>
        <input
          type="text"
          maxlength="20"
          bind:value={appState.payment.accountNumber}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">A/C Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.accountName}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.bank}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.ifsc}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.swiftCode}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
        />
      </div>
    </div>
  </div>

  <!-- Download Button -------------------------------------------------->

  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-[#E2E2E2] text-[#333] font-semibold flex items-center gap-2 hover:bg-[#CFCFCF] transition-transform duration-150"
    >
      <Download class="w-5 h-5" />
      <span>Download Invoice</span>
    </button>
  </div>
</div>

<style>
  .toggle-switch {
    width: 36px;
    height: 18px;
    background-color: #e2e2e2;
    border-radius: 20px;
    position: relative;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .toggle-switch:before {
    content: '';
    position: absolute;
    top: 1px;
    left: 1px;
    width: 16px;
    height: 16px;
    background-color: #8eaccd;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #8eaccd;
  }

  .toggle-switch.active:before {
    transform: translateX(18px);
    background-color: #feffd9;
  }
</style>
