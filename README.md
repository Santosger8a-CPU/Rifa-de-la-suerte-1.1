<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Rifa de la Suerte</title>
<script src="https://kit.fontawesome.com/a076d05399.js" crossorigin="anonymous"></script>
<link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
<style>
.number-item.selected{background-color:#4f46e5;color:white;border-color:#4338ca;}
.number-item.sold{background-color:#e5e7eb;color:#9ca3af;cursor:not-allowed;}
.ticket{display:inline-block;width:60px;height:90px;background:linear-gradient(to bottom,#6366f1,#8b5cf6);color:white;font-weight:bold;border-radius:8px;margin:2px;display:flex;align-items:center;justify-content:center;}
</style>
</head>
<body class="bg-gray-100">

<div class="container mx-auto p-6">

    <!-- Información Bancaria -->
    <div class="mt-12 bg-white rounded-2xl shadow-xl p-8">
        <h2 class="text-2xl font-bold text-gray-800 mb-6 flex items-center">
            <i class="fas fa-university mr-3 text-indigo-600"></i>
            Datos para transferencia bancaria / Flow
        </h2>
        <div class="space-y-3 text-gray-700">
            <p><i class="fas fa-check-circle text-green-500 mr-2"></i> Puedes pagar a través de Flow:</p>
            <p><a href="https://www.flow.cl/uri/JPVc0C1YX" target="_blank" class="text-blue-600 underline">Pagar con Flow</a></p>
        </div>
    </div>

    <!-- Sorteo -->
    <div class="mt-12 bg-white rounded-2xl shadow-xl p-8">
        <h2 class="text-2xl font-bold text-gray-800 mb-6 text-center">
            <i class="fas fa-trophy mr-3 text-yellow-500"></i>
            Premios del Sorteo
        </h2>
        <div class="grid md:grid-cols-3 gap-6 text-center text-white">
            <div class="bg-indigo-600 p-6 rounded-2xl">
                <div class="text-6xl font-bold mb-2">1°</div>
                <div class="text-xl font-bold mb-1">Primer Lugar</div>
                <div class="text-lg mb-4">$50.000</div>
            </div>
            <div class="bg-indigo-500 p-6 rounded-2xl">
                <div class="text-5xl font-bold mb-2">2°</div>
                <div class="text-xl font-bold mb-1">Segundo Lugar</div>
                <div class="text-lg mb-4">$30.000</div>
            </div>
            <div class="bg-indigo-400 p-6 rounded-2xl">
                <div class="text-4xl font-bold mb-2">3°</div>
                <div class="text-xl font-bold mb-1">Tercer Lugar</div>
                <div class="text-lg mb-4">$20.000</div>
            </div>
        </div>
    </div>

    <!-- Selección de números y registro -->
    <div class="mt-12 bg-white rounded-2xl shadow-xl p-8">
        <h2 class="text-2xl font-bold mb-6">Compra tus números</h2>
        <form id="registration-form" class="space-y-4">
            <div>
                <label class="block font-semibold">Cantidad de boletos:</label>
                <input type="number" id="quantity" value="1" min="1" max="10" class="border p-2 rounded w-24"/>
            </div>
            <div id="number-grid" class="grid grid-cols-10 gap-2"></div>

            <div class="grid md:grid-cols-3 gap-4">
                <div>
                    <label class="block font-semibold">Nombre:</label>
                    <input type="text" id="name" required class="border p-2 rounded w-full"/>
                </div>
                <div>
                    <label class="block font-semibold">Teléfono:</label>
                    <input type="text" id="phone" required class="border p-2 rounded w-full"/>
                </div>
                <div>
                    <label class="block font-semibold">Correo electrónico:</label>
                    <input type="email" id="email" required class="border p-2 rounded w-full"/>
                </div>
            </div>

            <div>
                <label class="block font-semibold mb-2">Método de pago:</label>
                <div class="flex gap-4">
                    <div class="payment-method border p-4 rounded cursor-pointer" data-payment="flow">
                        Flow
                    </div>
                </div>
            </div>

            <button type="submit" id="buy-button" class="bg-indigo-600 text-white px-6 py-3 rounded-lg mt-4 hover:bg-indigo-700 transition-colors">
                <i class="fas fa-shopping-cart mr-2"></i>Proceder al pago
            </button>
        </form>
    </div>

    <!-- Ticket -->
    <div id="ticket-container" class="hidden mt-12 max-w-4xl mx-auto bg-white p-8 rounded-2xl shadow-2xl">
        <div class="text-center mb-4 text-2xl font-bold text-green-600">¡Compra exitosa!</div>
        <div id="ticket-numbers" class="flex flex-wrap justify-center mb-4"></div>
        <p><strong>Nombre:</strong> <span id="ticket-name"></span></p>
        <p><strong>Teléfono:</strong> <span id="ticket-phone"></span></p>
        <p><strong>Email:</strong> <span id="ticket-email"></span></p>
        <p><strong>Total pagado:</strong> <span id="ticket-total"></span></p>
        <p><strong>Fecha del sorteo:</strong> <span id="ticket-draw-date"></span></p>
        <button id="new-purchase" class="mt-4 bg-indigo-600 text-white px-6 py-3 rounded-lg hover:bg-indigo-700">Comprar más números</button>
    </div>

</div>

<script>
const totalNumbers=100;
let soldNumbers=[];
let selectedNumbers=[];
let selectedPaymentMethod=null;
let purchases=[];
const pricePerTicket=2000;

function generateNumberGrid(){
    const grid=document.getElementById('number-grid');
    grid.innerHTML='';
    for(let i=1;i<=totalNumbers;i++){
        const div=document.createElement('div');
        div.textContent=i.toString().padStart(2,'0');
        div.className='number-item border p-2 text-center cursor-pointer';
        div.dataset.number=i;
        if(soldNumbers.includes(i)){
            div.classList.add('sold');
        }else{
            div.addEventListener('click',toggleNumberSelection);
        }
        grid.appendChild(div);
    }
}
function toggleNumberSelection(e){
    const number=parseInt(e.target.dataset.number);
    const quantity=parseInt(document.getElementById('quantity').value);
    if(selectedNumbers.includes(number)){
        selectedNumbers=selectedNumbers.filter(n=>n!==number);
        e.target.classList.remove('selected');
    }else{
        if(selectedNumbers.length<quantity){
            selectedNumbers.push(number);
            e.target.classList.add('selected');
        }else{
            alert('Ya seleccionaste la cantidad de números que compraste.');
        }
    }
}
document.querySelectorAll('.payment-method').forEach(m=>{
    m.addEventListener('click',function(){
        document.querySelectorAll('.payment-method').forEach(x=>x.classList.remove('selected'));
        this.classList.add('selected');
        selectedPaymentMethod=this.dataset.payment;
    });
});

document.getElementById('registration-form').addEventListener('submit',function(e){
    e.preventDefault();
    const quantity=parseInt(document.getElementById('quantity').value);
    if(selectedNumbers.length!==quantity){alert('Selecciona exactamente '+quantity+' números');return;}
    if(!selectedPaymentMethod){alert('Selecciona método de pago');return;}
    const name=document.getElementById('name').value;
    const phone=document.getElementById('phone').value;
    const email=document.getElementById('email').value;
    let total=(quantity===3)?5000:quantity*pricePerTicket;
    // Simular pago Flow
    window.open('https://www.flow.cl/uri/JPVc0C1YX','_blank');

    selectedNumbers.forEach(n=>{if(!soldNumbers.includes(n)) soldNumbers.push(n);});
    const now=new Date();
    const purchase={name,phone,email,numbers:selectedNumbers.join(', '),total,date:now.toLocaleDateString('es-ES')+' '+now.toLocaleTimeString('es-ES',{hour:'2-digit',minute:'2-digit'})};
    purchases.push(purchase);
    updateTicket(name,phone,email,total);
    selectedNumbers=[];
    selectedPaymentMethod=null;
    generateNumberGrid();
});

function updateTicket(name,phone,email,total){
    document.getElementById('ticket-name').textContent=name;
    document.getElementById('ticket-phone').textContent=phone;
    document.getElementById('ticket-email').textContent=email;
    document.getElementById('ticket-total').textContent='$'+total;

    const now=new Date();
    const drawDate=new Date(now);
    drawDate.setDate(drawDate.getDate()+7);
    drawDate.setHours(20,0,0,0);
    document.getElementById('ticket-draw-date').textContent=drawDate.toLocaleDateString('es-ES',{weekday:'long',year:'numeric',month:'long',day:'numeric',hour:'2-digit',minute:'2-digit'});

    const container=document.getElementById('ticket-numbers');
    container.innerHTML='';
    purchases[purchases.length-1].numbers.split(',').forEach(n=>{
        const div=document.createElement('div');
        div.textContent=n;
        div.className='ticket';
        container.appendChild(div);
    });

    document.getElementById('ticket-container').classList.remove('hidden');
}

document.getElementById('new-purchase').addEventListener('click',function(){
    document.getElementById('ticket-container').classList.add('hidden');
});
document.addEventListener('DOMContentLoaded',function(){generateNumberGrid();});
</script>
</body>
</html>
